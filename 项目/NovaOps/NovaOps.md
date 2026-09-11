---
title: "NovaOps"
date: 2026-08-24
section: "Projects"
source: http://120.77.152.123:8088/posts/novaops
tags:
  - "Projects"
  - "博客迁移"
---
# NovaOps 环境配置经验总结

> 日期：2026-08-24 场景：Windows 本地开发，通过 MobaXterm 连接 Linux 服务器；MySQL 和 Qdrant 运行在服务器 Docker 中，前后端运行在本地。

## 1. 最终采用的联调架构

```text
Windows 本地
├─ Vue/Vite 前端          http://127.0.0.1:5173
├─ Spring Boot 后端       http://127.0.0.1:8080
├─ SSH 本地端口 3307 ─────────────┐
└─ SSH 本地端口 6333 ───────────┐ │
                               │ │
Linux 服务器                   │ │
├─ Docker MySQL 8.4  127.0.0.1:3307
└─ Docker Qdrant     127.0.0.1:6333
```

关键原则：

- 数据库和向量库放服务器，开发时不用在 Windows 重复安装服务。
- Docker 端口只绑定服务器的 `127.0.0.1`，不直接暴露公网。
- Windows 通过 MobaXterm SSH Tunnel 访问服务器服务。
- 后端仍在本地运行，因此连接地址写本地隧道端口 `127.0.0.1:3307` 和 `127.0.0.1:6333`。
- 密码、JWT 密钥和模型 API Key 放环境变量，不提交 Git。

## 2. 服务器 Docker 配置

### 2.1 目录约定

服务器项目目录：

```bash
cd ~/novaops
```

初始化 SQL 应位于：

```text
~/novaops/backend/sql/novaops_init.sql
```

### 2.2 Compose 示例

服务器已有 1Panel MariaDB 占用 `127.0.0.1:3306`，因此 NovaOps MySQL 映射到服务器 `3307`：

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: novaops-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: novaops
      MYSQL_USER: novaops
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      TZ: Asia/Shanghai
    ports:
      - "127.0.0.1:3307:3306"
    volumes:
      - novaops-mysql-data:/var/lib/mysql
      - ./backend/sql/novaops_init.sql:/docker-entrypoint-initdb.d/01-init.sql:ro
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_general_ci

  qdrant:
    image: qdrant/qdrant:v1.15.4
    container_name: novaops-qdrant
    restart: unless-stopped
    ports:
      - "127.0.0.1:6333:6333"
      - "127.0.0.1:6334:6334"
    volumes:
      - novaops-qdrant-data:/qdrant/storage

volumes:
  novaops-mysql-data:
  novaops-qdrant-data:
```

服务器 `.env` 示例：

```dotenv
MYSQL_ROOT_PASSWORD=替换为高强度root密码
MYSQL_PASSWORD=替换为NovaOps数据库用户密码
```

建议限制权限：

```bash
chmod 600 .env
```

注意：仓库当前的 `docker-compose.yml` 可能只包含 Qdrant；服务器上的完整 Compose 是本次部署使用的独立配置，不要在未核对差异时直接覆盖。

### 2.3 启动和检查

```bash
docker compose config
docker compose up -d
docker compose ps
```

检查 Qdrant：

```bash
curl http://127.0.0.1:6333/collections
```

正常响应类似：

```json
{"result":{"collections":[]},"status":"ok"}
```

检查 MySQL 表：

```bash
docker compose exec mysql sh -c \
  'mysql -unovaops -p"$MYSQL_PASSWORD" --default-character-set=utf8mb4 -e "use novaops; show tables;"'
```

## 3. 今天遇到的服务器问题

### 3.1 Compose YAML 被 HTML 转义字符污染

错误：

```text
yaml: line 2, column 8: mapping values are not allowed in this context
```

文件中实际出现了：

```text
services
&#x20; mysql:
container\_name
```

这里有三个问题：

- `services` 后缺少冒号，应为 `services:`。
- `&#x20;` 是网页中的空格实体，不是 YAML 缩进。
- `\_` 是 Markdown 转义，不是合法配置字段，应为普通 `_`。

检查隐藏字符和真实行内容：

```bash
sed -n '1,20l' docker-compose.yml
nl -ba docker-compose.yml | head -30
```

经验：不要从经过富文本或 HTML 转义的页面直接复制 YAML。优先通过 MobaXterm 文件面板上传，或在纯文本编辑器中创建；写完先执行 `docker compose config`。

### 3.2 3306 端口冲突

错误：

```text
Bind for 127.0.0.1:3306 failed: port is already allocated
```

排查：

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}\t{{.Ports}}'
```

发现 1Panel MariaDB 已占用 `127.0.0.1:3306`，因此把 NovaOps 改为：

```yaml
ports:
  - "127.0.0.1:3307:3306"
```

注意两侧端口含义：

```text
服务器宿主机 3307 -> NovaOps MySQL 容器 3306
```

本地后端 JDBC 也必须连接隧道后的 `3307`，不能继续写 `3306`。

### 3.3 SQL 文件正常，但数据库中文已乱码

初始化 SQL 源文件中的中文是正常的，但表中出现了类似：

```text
ç®¡ç�†å‘˜
```

说明问题发生在“SQL 客户端读取文件并写入数据库”的链路中，而不是前端字体问题。

仅设置 MySQL 服务端字符集仍不够。建议在初始化 SQL 开头明确增加：

```sql
SET NAMES utf8mb4;
```

导入时也明确指定：

```bash
mysql --default-character-set=utf8mb4 -unovaops -p novaops \
  < backend/sql/novaops_init.sql
```

检查实际存储字节：

```sql
select id, name, hex(name), char_length(name), length(name)
from sys_role;
```

重要结论：修改字符集配置不会自动修复已经写坏的数据。已有乱码数据需要从正确的初始化 SQL 重新导入，或写专门的数据修复脚本。

### 3.4 mysqldump 缺少 PROCESS 权限

错误：

```text
Access denied; you need (at least one of) the PROCESS privilege(s)
when trying to dump tablespaces
```

普通业务账号备份时增加 `--no-tablespaces`：

```bash
docker compose exec -T mysql sh -c \
  'mysqldump -unovaops -p"$MYSQL_PASSWORD" --default-character-set=utf8mb4 --no-tablespaces novaops' \
  > novaops-before-fix.sql
```

备份完成后至少检查：

```bash
ls -lh novaops-before-fix.sql
head -20 novaops-before-fix.sql
```

## 4. MobaXterm SSH 隧道

在 MobaXterm 中打开 `Tunneling`，分别创建两个 Local port forwarding：

| 用途 | 本地端口 | 远端地址 | 远端端口 |
| --- | --- | --- | --- |
| MySQL | 3307 | 127.0.0.1 | 3307 |
| Qdrant HTTP | 6333 | 127.0.0.1 | 6333 |

两条隧道都通过同一台 Linux 服务器的 SSH 登录信息连接。

也可以用命令行表达同一件事：

```powershell
ssh -L 3307:127.0.0.1:3307 `
    -L 6333:127.0.0.1:6333 `
    root@你的服务器地址
```

在 Windows PowerShell 验证：

```powershell
Test-NetConnection 127.0.0.1 -Port 3307
Test-NetConnection 127.0.0.1 -Port 6333
```

两项的 `TcpTestSucceeded` 都应为 `True`。

检查 Qdrant：

```powershell
Invoke-RestMethod http://127.0.0.1:6333/collections
```

注意：隧道依赖 SSH 会话。MobaXterm 关闭或隧道停止后，本地后端会立即失去数据库和 Qdrant 连接。

## 5. Windows 本地开发环境

### 5.1 Maven

Maven 3.9.16 解压后，把它的 `bin` 目录加入用户 `Path` 即可，例如：

```text
E:\tools\apache-maven-3.9.16\bin
```

重新打开 PowerShell 和 IDE 后验证：

```powershell
mvn -v
```

本项目后端使用仓库内的 Maven settings：

```powershell
cd E:\project\Lan\NovaOps\backend
mvn -gs mvn-settings.xml test
mvn -gs mvn-settings.xml spring-boot:run
```

如果命令行能识别 Maven、IDE 终端不能识别，通常是 IDE 尚未重启，仍持有旧的 `Path`。

### 5.2 Node.js

安装依赖时出现：

```text
SyntaxError: Unexpected token '.'
SyntaxError: Unexpected token '='
```

报错位置包含：

```js
options?.cause
o.scripts ||= {}
```

这不是 MSW 或 Husky 本身损坏，而是 Node.js 版本过旧，不支持可选链或逻辑赋值语法。

处理顺序：

```powershell
node -v
npm -v
```

升级到现代 Node.js LTS 后，重新打开终端，再重新安装：

```powershell
Remove-Item -Recurse -Force node_modules
npm ci
```

不要在旧 Node 生成的半成品 `node_modules` 上继续排查前端业务代码。

## 6. 本地后端配置

### 6.1 推荐使用当前 PowerShell 会话环境变量

```powershell
$env:NOVAOPS_DB_URL = "jdbc:mysql://127.0.0.1:3307/novaops?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true&characterEncoding=utf8"
$env:NOVAOPS_DB_USERNAME = "novaops"
$env:NOVAOPS_DB_PASSWORD = "服务器.env中的MYSQL_PASSWORD"

$env:QDRANT_BASE_URL = "http://127.0.0.1:6333"
$env:QDRANT_COLLECTION = "novaops_kb"

$env:SILICONFLOW_API_KEY = "你的SiliconFlow API Key"
$env:SILICONFLOW_BASE_URL = "https://api.siliconflow.cn"
$env:SILICONFLOW_CHAT_MODEL = "Qwen/Qwen3-8B"
$env:SILICONFLOW_EMBEDDING_MODEL = "BAAI/bge-m3"

$env:NOVAOPS_JWT_SECRET = ([guid]::NewGuid().ToString("N") + [guid]::NewGuid().ToString("N"))
$env:SPRING_PROFILES_ACTIVE = "local"
```

验证 JWT 长度：

```powershell
$env:NOVAOPS_JWT_SECRET.Length
```

结果为 `64` 正常，项目最低要求是 32 字节。

### 6.2 是否写系统环境变量

- `SPRING_PROFILES_ACTIVE`、数据库 URL、用户名等稳定配置可以写用户环境变量。
- 数据库密码、JWT 和 API Key 更建议放当前终端、IDE Run Configuration 或不提交的本地配置中。
- 一般不必写“系统环境变量”；用户环境变量已经足够，权限范围也更小。
- 修改持久环境变量后必须重新打开 PowerShell、IDE 和后端进程。
- 不要把真实值写进 README、Git 跟踪的 YAML、截图或提交记录。

### 6.3 application-local.yml

可以复制：

```text
backend/src/main/resources/application-local.yml.example
```

为：

```text
backend/src/main/resources/application-local.yml
```

该文件已经被 `.gitignore` 忽略。建议 YAML 只引用环境变量：

```yaml
spring:
  datasource:
    url: ${NOVAOPS_DB_URL}
    username: ${NOVAOPS_DB_USERNAME}
    password: ${NOVAOPS_DB_PASSWORD}

app:
  security:
    jwt-secret: ${NOVAOPS_JWT_SECRET}
  kb:
    qdrant-base-url: ${QDRANT_BASE_URL:http://127.0.0.1:6333}
```

Spring AI 和模型配置已在 `application.yml` 中读取 `SILICONFLOW_*` 环境变量，不需要重复写密钥。

## 7. 前后端启动顺序

### 7.1 每次联调前

1. 确认服务器容器正常：`docker compose ps`。
2. 启动 MobaXterm 的 3307、6333 隧道。
3. 用 `Test-NetConnection` 检查两个端口。
4. 打开新的 PowerShell，使环境变量生效。

### 7.2 启动后端

```powershell
cd E:\project\Lan\NovaOps\backend
mvn -gs mvn-settings.xml spring-boot:run
```

默认地址：

```text
http://127.0.0.1:8080
```

修改 YAML 或环境变量后，需要重启 Spring Boot；仅刷新浏览器不会让后端重新读取配置。

### 7.3 启动前端

另开 PowerShell：

```powershell
cd E:\project\Lan\NovaOps
npm run dev
```

默认地址：

```text
http://127.0.0.1:5173
```

开发环境使用：

```dotenv
VITE_API_BASE_URL=/api
VITE_ENABLE_MOCK=partial
```

`partial` 模式下认证、Agent、工单和知识库文档等接口会访问真实 Java 后端；排查问题时必须确认当前 mock 模式，不能把 full mock 的结果当成真实后端状态。

## 8. 推荐验收清单

```powershell
# 1. SSH 隧道
Test-NetConnection 127.0.0.1 -Port 3307
Test-NetConnection 127.0.0.1 -Port 6333

# 2. Qdrant
Invoke-RestMethod http://127.0.0.1:6333/collections

# 3. 后端测试
cd E:\project\Lan\NovaOps\backend
mvn -gs mvn-settings.xml test

# 4. 前端检查
cd E:\project\Lan\NovaOps
npm run lint
npm run build
npm run dev
```

浏览器联调至少检查：

- 能使用正确租户和账号登录。
- 页面中文没有乱码。
- 后端可以查询 MySQL 中的初始化数据。
- 知识库文件可以上传并完成向量化。
- `你好` 等安全寒暄能正常回复。
- 企业问题走 RAG，有资料时返回引用，无资料时明确拒答。
- 关闭 MobaXterm 隧道后能观察到连接失败，重新打开后重启后端可恢复。

## 9. 其他小坑

### Commitlint 认为 type 和 subject 为空

如果提交内容看起来是：

```text
"fix: 修复数据库初始化编码"
```

但 Commitlint 报：

```text
subject may not be empty
type may not be empty
```

通常是提交信息被额外引号、转义字符、编码或编辑器调用方式污染。可以直接在终端提交：

```powershell
git commit -m "fix(db): 修复初始化编码"
```

提交前可先检查：

```powershell
git status
git diff --cached
```

## 10. 最重要的经验

1. 先验证链路，再启动应用：容器、端口、SSH 隧道、数据库、Qdrant 应逐层检查。
2. `127.0.0.1` 指向“当前运行该进程的机器”：服务器容器映射和 Windows 本地后端不能混为一谈。
3. 端口冲突不要强行停掉现有业务数据库，给新服务换宿主机端口即可。
4. YAML 只接受真实空格和字段字符，网页实体与 Markdown 转义会直接破坏配置。
5. 服务端 `utf8mb4` 不代表导入客户端一定使用 UTF-8；初始化 SQL 和导入命令都应明确字符集。
6. 源 SQL 正常不代表库中数据正常，使用 `HEX()` 查看真实存储字节。
7. 前端依赖出现新语法解析错误时先检查 Node 版本，不要先怀疑业务代码。
8. 密钥长度 64 没问题，但密钥本身不能进入仓库。
9. 配置变化后重启对应进程；环境变量不会自动注入已经运行的后端。
10. 最终以运行结果为准，而不是以“配置文件看起来正确”为准。
