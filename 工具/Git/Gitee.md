1.Gitee：
创建仓库：
![[Pasted image 20260126160401.png]]

配置SSH公钥到Gitee中
![[Pasted image 20260126160347.png]]

再git clone克隆仓库（之前因为没有在Gitee设置私钥，报错了）：
![[Pasted image 20260126160502.png]]
一切正常，克隆了一个新的空仓库。

`git push`出现报错：
![[Pasted image 20260126161012.png]]
解释：
**本地分支叫 `main`，但它的 upstream（跟踪的远程分支）指向了 `origin/master`**，名字不一致，所以 `git push`（默认 `simple`）不敢直接推，怕推错分支。

解决：
 把本地 main 绑定到远程 main（以后直接 `git push` 就行）
`git push -u origin main`
- `-u` 等价于 `--set-upstream`，意思是：**把你当前本地分支设置为跟踪（upstream）某个远程分支**。
- `origin`：远程仓库名
- `main`：你要推送的 **本地分支名**（并且默认推到远程同名 `main`）
- `-u`：顺便把本地 `main` 的 upstream 设为 `origin/main`，实现对远程仓库的main分支的追踪。

但仍然报错：
![[Pasted image 20260126161744.png]]
错误原因：
 现在这个 `main` 分支 **还没有任何 commit**（“空分支 / unborn branch”）
Git 只有在分支上至少有一个提交时，`refs/heads/main` 才真正存在；否则你 push 的时候就会说“找不到 main 这个 refspec”。

解决：先做一次提交，再 push，解决：
![[Pasted image 20260126161857.png]]
