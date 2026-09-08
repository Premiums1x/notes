配置过程中出现的问题：
1. `~/.ssh` 里**只有 known_hosts**，没有 `id_ed25519` / `id_rsa` 之类的 **key 文件**
    
2. `ssh-add -l` 报 `Could not open a connection to your authentication agent.` → **ssh-agent 没启动**
    
3. 所以 `ssh -T git@github.com` 只能 `Permission denied (publickey)`（因为根本没有可用的公钥给 GitHub 验证）
    
也就是说：**你现在电脑上还没生成 SSH key（或至少 Git Bash 这套环境看不到），并且 agent 也没开。**

流程：
## 1）生成 SSH key（推荐 ed25519）

在 Git Bash 里：

`ssh-keygen -t ed25519 -C "2020829761@qq.com"`

接下来它会问保存路径，例如：  
`Enter file in which to save the key (/c/Users/takumi/.ssh/id_ed25519):`  
直接回车就行。

再问 passphrase（密码）：

- 直接回车（无密码，方便）
    
- 或者设一个（更安全，但每次可能要输/缓存）
    

生成完后，检查一下：

`ls -al ~/.ssh`

你应该能看到：

- `id_ed25519`（私钥）
    
- `id_ed25519.pub`（公钥）
    

---

## 2）启动 ssh-agent（解决你 ssh-add 的报错）

在 Git Bash 里执行：

`eval "$(ssh-agent -s)"`

然后把私钥加入 agent：

`ssh-add ~/.ssh/id_ed25519`

再确认是否加载成功：

`ssh-add -l`

这次应该会列出一条 key，而不是报错。

> 如果 `eval "$(ssh-agent -s)"` 仍然不行，告诉我报错内容，我再给你 Windows 服务那套（PowerShell `Start-Service ssh-agent`）的方案。

---

## 3）把公钥加到 GitHub

把公钥打印出来：

`cat ~/.ssh/id_ed25519.pub`

复制整行（以 `ssh-ed25519` 开头那一整串），到 GitHub：

**Settings → SSH and GPG keys → New SSH key → 粘贴 → Add**

---

## 4）测试 SSH 登录

`ssh -T git@github.com`

成功一般会提示类似：`Hi Premiums11x! You've successfully authenticated...`

---

## 5）再 clone

`git clone git@github.com:Premiums11x/gitLearning.git`


配置过程代码：
![[Pasted image 20260126012843.png]]
![[Pasted image 20260126012900.png]]

进入仓库中新建文件测试：
![[Pasted image 20260126013022.png]]
此时只是在本地仓库进行操作，还未推送到远程仓库中。
