---
title: "NovaOps 远程 MySQL 与 Qdrant SSH 隧道配置"
date: 2026-08-26
section: "Debug"
source: http://120.77.152.123:8088/posts/%e8%bf%9c%e7%a8%8b%20MySQL%20%e4%b8%8e%20Qdrant%20SSH%20%e9%9a%a7%e9%81%93
tags:
  - "Debug"
  - "博客迁移"
---
# NovaOps SSH 隧道配置与排障总结

> 日期：2026-08-25 场景：NovaOps 后端运行在本地 Windows，MySQL 和 Qdrant 运行在云服务器的 Docker 容器中。

## 一、整体网络结构

由于服务器上的 MySQL 和 Qdrant 只绑定在服务器的 `127.0.0.1` 上，因此它们无法通过公网 IP 直接访问。

本地 NovaOps 后端通过 SSH 隧道访问这两个服务。

### 1. Qdrant 访问链路

```text
本地 Windows 上的 NovaOps 后端
        ↓
本机 127.0.0.1:16333
        ↓
SSH 加密隧道
        ↓
云服务器 127.0.0.1:6333
        ↓
Qdrant 容器 6333
```

对应的 SSH 转发配置为：

```text
127.0.0.1:16333 → 服务器 127.0.0.1:6333
```

### 2. MySQL 访问链路

```text
本地 Windows 上的 NovaOps 后端
        ↓
本机 127.0.0.1:13307
        ↓
SSH 加密隧道
        ↓
云服务器 127.0.0.1:3307
        ↓
MySQL 容器 3306
```

对应的 SSH 转发配置为：

```text
127.0.0.1:13307 → 服务器 127.0.0.1:3307
```

其中：

- `13307` 是本地 Windows 上的 SSH 监听端口。
- `3307` 是云服务器宿主机上的 Docker 映射端口。
- `3306` 是 MySQL 容器内部监听的端口。
- `16333` 是本地 Windows 上的 SSH 监听端口。
- `6333` 是云服务器上 Qdrant 的访问端口。

## 二、建立 SSH 隧道

在本地 Windows 的 PowerShell 中执行：

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" -N `
  -o ExitOnForwardFailure=yes `
  -o ServerAliveInterval=30 `
  -o ServerAliveCountMax=3 `
  -L 127.0.0.1:13307:127.0.0.1:3307 `
  -L 127.0.0.1:16333:127.0.0.1:6333 `
  root@120.77.152.123
```

执行后，如果命令没有输出并且一直不返回，通常表示 SSH 隧道已经成功建立，当前正在前台运行。

此时需要注意：

- 不要关闭当前 PowerShell 窗口。
- 不要在该窗口中继续输入其他命令。
- 按下 `Ctrl+C` 可以关闭 SSH 隧道。
- PowerShell 窗口被关闭后，隧道也会随之断开。

### SSH 参数说明

| 参数 | 作用 |
| --- | --- |
| `-i` | 指定用于登录服务器的 SSH 私钥 |
| `-N` | 只建立 SSH 隧道，不打开远程终端 |
| `-L` | 建立本地端口转发 |
| `ExitOnForwardFailure=yes` | 如果本地端口绑定失败，则立即退出并显示错误 |
| `ServerAliveInterval=30` | 每隔 30 秒向服务器发送一次 SSH 保活消息 |
| `ServerAliveCountMax=3` | 连续 3 次未收到服务器响应后断开连接 |

## 三、本地端口转发的含义

SSH 本地端口转发的基本格式为：

```text
-L 本地监听地址:本地监听端口:远程目标地址:远程目标端口
```

例如：

```text
-L 127.0.0.1:16333:127.0.0.1:6333
```

可以拆分为：

| 部分 | 含义 |
| --- | --- |
| 第一个 `127.0.0.1` | SSH 在本地 Windows 上监听的地址 |
| `16333` | SSH 在本地 Windows 上监听的端口 |
| 第二个 `127.0.0.1` | SSH 登录服务器后，从服务器视角访问的目标地址 |
| `6333` | 云服务器上的目标端口 |

完整含义是：

> SSH 在本地 Windows 的 `127.0.0.1:16333` 上监听连接，并将收到的请求通过 SSH 加密连接转发到云服务器的 `127.0.0.1:6333`。

因此，本地程序只需要访问：

```text
http://127.0.0.1:16333
```

SSH 就会把请求转发到云服务器上的 Qdrant。

同理：

```text
-L 127.0.0.1:13307:127.0.0.1:3307
```

表示：

> SSH 在本地 Windows 的 `127.0.0.1:13307` 上监听连接，并将请求转发到云服务器的 `127.0.0.1:3307`。

## 四、配置 NovaOps 后端

SSH 隧道建立成功后，需要让本地运行的 NovaOps 后端访问本地隧道端口，而不是直接访问云服务器端口。

在 PowerShell 中设置以下环境变量：

```powershell
$env:QDRANT_BASE_URL="http://127.0.0.1:16333"

$env:NOVAOPS_DB_URL="jdbc:mysql://127.0.0.1:13307/novaops?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true&characterEncoding=utf8"
```

### 配置含义

Qdrant 的连接地址为：

```text
http://127.0.0.1:16333
```

MySQL 的连接地址为：

```text
jdbc:mysql://127.0.0.1:13307/novaops
```

这里填写的是本地 SSH 隧道监听端口，而不是云服务器端口：

| 服务 | 本地后端应访问的地址 | 云服务器实际端口 |
| --- | --- | --- |
| Qdrant | `127.0.0.1:16333` | `127.0.0.1:6333` |
| MySQL | `127.0.0.1:13307` | `127.0.0.1:3307` |

### 环境变量的作用范围

PowerShell 中通过 `$env:` 设置的环境变量，只对以下对象有效：

- 当前 PowerShell 窗口。
- 从当前 PowerShell 窗口启动的子进程。

因此，设置环境变量后，应该在同一个 PowerShell 窗口中启动 NovaOps 后端。

如果通过 IntelliJ IDEA 等 IDE 启动后端，则需要把环境变量添加到 IDE 的运行配置中。仅在另一个 PowerShell 窗口中设置变量，不会影响已经启动的 IDE。

## 五、验证 SSH 隧道

不要只根据 MobaXterm 中是否存在隧道配置判断隧道状态。

正确的判断方式是：

1. 检查本地端口是否处于监听状态。
2. 实际请求转发后的服务。
3. 确认后端连接的端口与隧道监听端口一致。

### 1. 检查本地端口监听状态

在另一个 PowerShell 窗口中执行：

```powershell
Get-NetTCPConnection -LocalPort 16333 -State Listen
Get-NetTCPConnection -LocalPort 13307 -State Listen
```

如果能够查询到对应记录，说明本机上有程序正在监听这些端口。

正常情况下，监听程序应该是 `ssh.exe`。

可以进一步查看对应进程：

```powershell
Get-NetTCPConnection -LocalPort 16333 -State Listen |
  Select-Object LocalAddress, LocalPort, State, OwningProcess

Get-NetTCPConnection -LocalPort 13307 -State Listen |
  Select-Object LocalAddress, LocalPort, State, OwningProcess
```

然后根据 `OwningProcess` 查询进程：

```powershell
Get-Process -Id <进程ID>
```

### 2. 验证 Qdrant

执行：

```powershell
curl.exe --noproxy "*" http://127.0.0.1:16333/readyz
```

其中，`--noproxy "*"` 表示本次请求不经过系统代理，避免本地代理软件干扰对 `127.0.0.1` 的访问。

成功时应返回：

```text
all shards are ready
```

也可以访问 Qdrant 根路径：

```powershell
curl.exe --noproxy "*" http://127.0.0.1:16333/
```

如果返回 Qdrant 的名称和版本信息，也说明转发成功。

### 3. 验证 MySQL 端口

执行：

```powershell
Test-NetConnection 127.0.0.1 -Port 13307
```

成功时应显示：

```text
TcpTestSucceeded : True
```

这只能说明本地 `13307` 端口可以建立 TCP 连接。

如果还需要确认数据库账号、密码、数据库名称和 JDBC 参数是否正确，应通过 MySQL 客户端或直接启动 NovaOps 后端进行验证。

## 六、本次问题的原因

最初的 Qdrant 配置为：

```text
QDRANT_BASE_URL=http://localhost:6333
```

但是 NovaOps 后端运行在本地 Windows，因此这里的 `localhost` 指向本地 Windows，而不是云服务器。

如果本地 Windows 的 `6333` 端口没有正确建立 SSH 转发，NovaOps 就无法通过该地址访问云服务器上的 Qdrant。

随后，在尝试让 SSH 监听本地 `6333` 端口时出现：

```text
bind [127.0.0.1]:6333: Permission denied
```

这个错误表示：

> SSH 无法在本地 Windows 上绑定并监听 `127.0.0.1:6333`。

该错误发生在本地端口绑定阶段，因此与云服务器防火墙无关，也不是 Qdrant 容器本身导致的。

最终改用本地端口 `16333`：

```text
本地 127.0.0.1:16333 → 云服务器 127.0.0.1:6333
```

SSH 隧道成功建立，NovaOps 后端也应相应改为访问：

```text
http://127.0.0.1:16333
```

## 七、正确理解 localhost

`localhost` 或 `127.0.0.1` 表示当前程序所在网络环境中的本机。

不同环境中的 `localhost` 并不是同一个位置。

| 程序运行位置 | `localhost` 指向 |
| --- | --- |
| 本地 Windows 程序 | 本地 Windows |
| 云服务器宿主机程序 | 云服务器宿主机 |
| Docker 容器内的程序 | 当前 Docker 容器自身 |
| WSL 内的程序 | 当前 WSL 网络环境 |

例如：

```text
NovaOps 后端运行在本地 Windows
```

那么它访问：

```text
localhost:6333
```

实际访问的是：

```text
本地 Windows 的 6333 端口
```

它不会自动访问云服务器的 `6333` 端口。

如果希望本地程序通过 `localhost` 访问云服务器服务，就必须通过 SSH 隧道、VPN、反向代理或其他网络转发机制建立连接。

## 八、Docker 端口映射的含义

MySQL 的访问链路中存在两层端口：

```text
云服务器宿主机 127.0.0.1:3307
        ↓
Docker 端口映射
        ↓
MySQL 容器 3306
```

类似的 Docker 端口映射配置为：

```text
127.0.0.1:3307:3306
```

其含义是：

| 部分 | 含义 |
| --- | --- |
| `127.0.0.1` | 只允许通过服务器本机访问 |
| `3307` | 云服务器宿主机端口 |
| `3306` | MySQL 容器内部端口 |

因此，在云服务器宿主机上访问：

```text
127.0.0.1:3307
```

Docker 会把连接转发给 MySQL 容器的：

```text
3306
```

SSH 隧道不需要直接连接容器内部的 `3306`，只需要连接云服务器宿主机已经映射出来的 `3307`。

完整链路为：

```text
本地 NovaOps
  → 本地 127.0.0.1:13307
  → SSH 隧道
  → 云服务器 127.0.0.1:3307
  → Docker 端口映射
  → MySQL 容器 3306
```

## 九、MobaXterm 隧道状态说明

MobaXterm 中存在 SSH 隧道配置记录，只能说明曾经配置过该隧道，并不能证明它当前仍在运行。

判断隧道是否正在运行，应以以下结果为准：

1. 本地端口是否处于 `Listen` 状态。
2. 监听端口的进程是否是 SSH 或 MobaXterm。
3. 实际请求能否通过隧道访问远端服务。
4. SSH 进程是否仍然存在。

例如：

```powershell
Get-NetTCPConnection -LocalPort 16333 -State Listen
```

如果没有任何输出，说明本机当前没有程序监听 `16333`，此时隧道通常没有运行。

## 十、常见错误及排查方向

| 错误信息 | 错误含义 | 常见原因 |
| --- | --- | --- |
| `Permission denied` | 系统拒绝本地端口绑定 | 端口被系统保留、安全软件拦截或当前进程无权绑定 |
| `Address already in use` | 本地端口已被占用 | 另一个 SSH 隧道或其他程序已经监听该端口 |
| `Connection refused` | 目标主机拒绝连接 | 远端目标端口没有服务监听，或 Docker 容器未启动 |
| `Connection timed out` | 连接超时 | 网络、路由、安全组或防火墙阻断 |
| `Connection reset` | 已建立的连接被重置 | 服务器、中间网络设备或目标程序主动断开连接 |
| `Could not resolve hostname` | 无法解析主机名 | SSH 地址填写错误或 DNS 解析失败 |
| `Host key verification failed` | 主机密钥验证失败 | 服务器密钥发生变化，或 `known_hosts` 中记录不匹配 |
| `Permission denied (publickey)` | SSH 公钥认证失败 | 私钥错误、公钥未配置或服务器权限设置错误 |
| `open failed: connect failed` | SSH 隧道已建立，但无法连接远端目标 | 远端地址或端口错误，或者远端服务没有监听 |

### 排查顺序

遇到连接问题时，可以按照以下顺序检查：

1. 确认云服务器上的 Docker 容器是否正在运行。
2. 确认服务器宿主机端口是否正在监听。
3. 确认服务器本机能否访问对应服务。
4. 确认 SSH 能否正常登录服务器。
5. 确认本地 SSH 隧道端口是否监听。
6. 确认本地请求能否通过隧道访问服务。
7. 确认 NovaOps 后端配置的地址和端口是否正确。
8. 确认代理软件没有拦截本地请求。

## 十一、SSH 公钥认证

本地私钥位置：

```text
C:\Users\takumi\.ssh\id_ed25519
```

服务器保存对应公钥的位置：

```text
/root/.ssh/authorized_keys
```

公钥认证的基本过程是：

1. 私钥保存在本地电脑。
2. 对应的公钥保存在服务器。
3. 服务器通过公钥验证本地客户端是否持有匹配的私钥。
4. 私钥不会通过网络发送给服务器。

服务器上的 SSH 目录权限应设置为：

```bash
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

如果权限设置不正确，SSH 服务可能会出于安全原因拒绝使用 `authorized_keys`。

### 安全注意事项

- 不要把私钥发送给任何人。
- 不要把私钥上传到服务器。
- 不要把私钥提交到 Git 仓库。
- 不要把数据库密码或 API Key 提交到 Git 仓库。
- 公钥可以部署到需要登录的服务器。
- 长期使用时，公钥认证比依赖服务器密码更安全、更方便。

## 十二、最终配置汇总

### SSH 隧道

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" -N `
  -o ExitOnForwardFailure=yes `
  -o ServerAliveInterval=30 `
  -o ServerAliveCountMax=3 `
  -L 127.0.0.1:13307:127.0.0.1:3307 `
  -L 127.0.0.1:16333:127.0.0.1:6333 `
  root@120.77.152.123
```

### NovaOps 环境变量

```powershell
$env:QDRANT_BASE_URL="http://127.0.0.1:16333"

$env:NOVAOPS_DB_URL="jdbc:mysql://127.0.0.1:13307/novaops?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true&characterEncoding=utf8"
```

### 隧道验证

```powershell
Get-NetTCPConnection -LocalPort 16333 -State Listen
Get-NetTCPConnection -LocalPort 13307 -State Listen

curl.exe --noproxy "*" http://127.0.0.1:16333/readyz

Test-NetConnection 127.0.0.1 -Port 13307
```

## 十三、核心经验总结

1. `localhost` 只表示程序当前所在环境，不等于远程服务器。
2. SSH 隧道命令应该在需要访问远程服务的本地电脑上执行。
3. `ssh -N` 执行后没有输出并且不返回，通常说明隧道正在前台运行。
4. SSH 隧道窗口关闭或 SSH 进程退出后，隧道也会断开。
5. 判断隧道状态时，应检查真实的端口监听状态和实际请求结果。
6. MobaXterm 中存在隧道配置记录，不代表隧道当前正在运行。
7. 本地端口和远端端口不需要相同。
8. 本地端口冲突或无法绑定时，可以改用其他未被占用的本地端口。
9. `bind` 错误发生在本地端口监听阶段，通常与服务器防火墙无关。
10. SSH 隧道可以安全访问仅绑定在服务器 `127.0.0.1` 上的服务。
11. Docker 宿主机端口和容器内部端口是两层不同的端口。
12. 后端必须连接本地 SSH 隧道端口，不能直接把本地 `localhost` 当作云服务器。
13. PowerShell 环境变量只对当前窗口及其启动的子进程有效。
14. 公钥认证比长期依赖服务器密码更适合开发和运维场景。
