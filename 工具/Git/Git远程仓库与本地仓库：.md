![[Pasted image 20260126013204.png]]
Git作为分布式版本控制系统，本地仓库和远程仓库是两个不同且相互独立的仓库
为了保证两个仓库之间的同步与更新：
`git push`:把在本地仓库进行的修改"推送"远程仓库
`git pull`：把在远程仓库进行的修改"拉取"到本地仓库


## 把远程仓库关联到本地仓库：
`git clone`:
![[Pasted image 20260126013454.png]]
截至目前，我们都是在对新建仓库所对应的“本地仓库”进行操作，如图：创建了一个hello.txt文件，但是这个新建文件并未推送到远程仓库中。

使用`git push`命令将我们在本地仓库的修改内容同步到远程仓库。
![[Pasted image 20260126013756.png]]

刷新github网页端：
![[Pasted image 20260126013920.png]]
可见我们在本地仓库的修改内容（新建了一个txt文件），已被同步到远程仓库。


## 把本地仓库关联到远程仓库：
`git remote add <shortname> <url>`
- 新建仓库：
- ![[Pasted image 20260126153236.png]]

执行：
```bash
git remote add origin git@github.com:Premiums11x/toTheRemoteTest.git
```

把本地repo仓库关联到远程仓库：
![[Pasted image 20260126153508.png]]

`git remote -v`：查看当前仓库对应的远程仓库的别名和地址
![[Pasted image 20260126153549.png]]

`git branch -M main`：设定当前分支为main，因为默认为main，此处我的分支为master，改为main：
![[Pasted image 20260126154329.png]]

`git push -u origin main`实际为：`git push -u origin main:main`: -u是把本地仓库和远程仓库关联，main:main是把本地分支main推送给远程仓库的main分支，名称相同就简写了。
`git push <远程仓库名> <本地分支名> <远程分支名>`
推送远程仓库：
![[Pasted image 20260126154346.png]]

## 模拟远程仓库有了修改，本地仓库拉取更新的情况：
README.md文件的内容会直接展示在github的仓库首页中，以帮助他人更好的了解仓库的作用等等，因此我们可以在文件中对仓库进行详细说明。
![[Pasted image 20260126154748.png]]
![[Pasted image 20260126155009.png]]

可见仓库变化：
![[Pasted image 20260126155041.png]]

而此时：远程仓库已经有了变化，本地仓库需要拉取更新：
拉取远程仓库的修改内容到本地：`git push 远程仓库名 <远程分支名>:<本地分支名>`
*执行pull操作时，git会自动为我们执行合并操作，远程仓库的修改与本地仓库的修改没有冲突时，操作成功，否则失败*

**此时使用fetch命令**：fetch命令只获取远程仓库的修改，并不会自动合并到本地仓库，需要自行手动合并。