起因：看UP主的终端美观且清晰，可以看到当前git分支：
![[Pasted image 20260121172243.png]]

而我的普通cmd：
![[Pasted image 20260121172308.png]]

美观步骤：
0）先确认你装了 Git
git --version

 1）安装 Windows Terminal / PowerShell / oh-my-posh
打开 **PowerShell（建议管理员）**，粘贴：
```
winget install Microsoft.WindowsTerminal
winget install Microsoft.PowerShell
winget install JanDeDobbeleer.OhMyPosh
```
![[Pasted image 20260121172530.png]]

2）安装“带图标字体”（不装就会乱码/缺符号）
`oh-my-posh font install`
选一个常用的
![[Pasted image 20260121172541.png]]
CascadiaCode Nerd Font


 3）让 Windows Terminal 用上这个字体
打开 **Windows Terminal** → `Settings(设置)` → 选（PowerShell）→ `Appearance(外观)` → `Font face(字体)`
![[Pasted image 20260121172651.png]]

 4）安装 **git 分支显示插件 posh-git**
PowerShell 输入：
`Install-Module posh-git -Scope CurrentUser -Force`
如果问你信不信任仓库，输入 `Y` 回车。
![[Pasted image 20260121172755.png]]

5）配置 PowerShell，让提示符变成彩色并显示分支
打开你的 PowerShell 配置文件：
`notepad $PROFILE`
![[Pasted image 20260121173015.png]]
把下面两行粘进去（如果文件是空的就直接粘）：
```
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\jandedobbeleer.omp.json" | Invoke-Expression
Import-Module posh-git
```
![[Pasted image 20260121173048.png]]
保存关闭。

 6）重开终端验证
关闭 Windows Terminal，再重新打开。  
进入你的 git 项目目录，比如：
`cd D:\learnGit`
应该会看到提示符里出现 **路径 + 分支名 main**，而且是彩色的：
![[Pasted image 20260121173147.png]]

config not found 出现了一点小问题:-
你系统里能调用的 `oh-my-posh` 路径在 `WindowsApps`，它更像一个“入口/别名”，**不自带 themes 目录**（至少你这套安装形态下没有）。
    
- 所以 `$env:POSH_THEMES_PATH` 是空的（或无效），你在 `$PROFILE` 里写的 `--config "$env:POSH_THEMES_PATH\xxx.omp.json"` 就会变成**指向一个不存在的文件**。
>**`$env:POSH_THEMES_PATH` 是环境变量**（PowerShell 的环境变量）
    
- `oh-my-posh` 找不到配置文件，于是提示符里显示 **CONFIG NOT FOUND**。
    
- 解决办法就是：**自己创建一个 `.omp.json` 主题配置文件**（你现在的 `mytheme.omp.json`），然后在 `$PROFILE` 里用 `--config` 指向它。这样不依赖内置 themes，也就不会再报错。
![[Pasted image 20260121175635.png]]