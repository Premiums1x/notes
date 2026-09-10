git是一种分布式版本控制系统，存放在仓库里的文件都有其历史版本。
>这里说一下*集中式版本控制系统*的原理：文件都存放在中央服务器中，用户手上的是个副本，使用只需要从中央服务器下载即可，如果需要修改则下载到本地对其修改后再次上传到中央服务器。
>>特点：使用方便，但中央服务器是单源的，用户依赖于其正常运行，一旦其故障或网络出现问题，所有人都无法正常工作了。
>

而分布式：每个人电脑上都有一个完整版本库，在本地修改无需网络，即使中央服务器故障也能正常使用。如需将修改内容分享给他人，只需同步一下仓库即可。


#Git使用方式 ：
1.命令行
2.图形化界面（GUI）
3.IDE插件

#git使用步骤： 
 1.配置用户名：
 `git config --global user.name "Lancer"`
 （默认省略）Local:本地生效
 --global:全局配置，所有仓库生效
 --system:系统配置，对所有用户生效

配置邮箱：
`git config --global user.email zengzhijiong2005@163.com`

 保存账号和密码：
` git config --global credential.helper store`

查看git配置信息：
`git config --global --list`

#2、创建版本库（仓库）Repository,管理本地代码：(把本地的一个文件目录变成git可以管理的仓库)
本地创建一个仓库：`git init`
远程服务器上克隆一个已存在的仓库:`git clone`

>本地创建git仓库：
创建一个空目录，使其变成git仓库：
![[Pasted image 20260121170540.png]]

我们使用美观后的powershell
查看文件：`ls`,因为.git文件是隐藏的，用`ls -hidden`或`ls -force`

不要轻易去动.git这个隐藏文件夹，容易导致当前文件目录不再是一个git仓库。

>`git init`可以加参数自定义一个仓库，如:`git init lancer`,则会新建一个lancer文件，而.git文件也存在该目录下，这种情况下lancer目录才是git仓库。
![[Pasted image 20260121181637.png]]

>利用远程服务器克隆一个git仓库：
>`git clone 远程服务器网站（github、gitee...）`
例：`git clone https://github.com/geekhall-laoyang/remote-repo.git`
![[Pasted image 20260121182004.png]]

#git中的数据管理：
>- 工作区（资源管理器中的本地文件） .git所在目录
>- 暂存区：临时存放、保存即将提交到git仓库的修改内容 .git/index
>- 本地仓库：通过`git init`命令创建的仓库，包含完整的项目历史和元数据，存储代码和
版本信息的主要位置  .git/objects

工作区进行修改后-----git add修改后的文件-->暂存区---git commit提交修改到--->本地仓库

*进行一次修改就执行一次add和commit会很繁琐，可以把修改的文件都add到暂存区后一次性commit到仓库里。*

Git中文件对应的状态
- 未跟踪(没有用Git管理的文件)
- 未修改（用Git管理但文件内容无变化）
- 已修改（文件内容变化但未添加到暂存区）
- 已暂存
![[Pasted image 20260121205403.png]]

#三个区域的差异一起汇总给你看 ：`git status`
> - **工作区（working tree）**：你本地文件实际长什么样
- **暂存区（staging area / index）**：下一次要提交的内容
- **HEAD（上一次提交）**：当前分支最新提交

>输出内容：
 1) `Changes to be committed`
✅ **暂存区里**已经 stage 好、下一次 commit 会带上的改动（相对 HEAD）

 2) `Changes not staged for commit`
✅ **工作区里**改了但还没 `git add` 的改动（相对暂存区/HEAD）

另外还有：
 `Untracked files`
✅ 工作区新文件，Git 还没追踪（没被 add 过）

 在仓库里新建一个txt文件：
![[Pasted image 20260121210644.png]]
可见gitFile1.txt是未追踪状态

*`git status -s`*:以简略模式查看仓库：
![[Pasted image 20260126003503.png]]
回显的最开始两列：第一列时暂存区状态，第二列是工作区状态。
以下是常见的状态标识：
![[Pasted image 20260126163716.png]]

#将文件添加到暂存区，等待后续提交操作： `git add`
 ![[Pasted image 20260121213610.png]]
 可见已被添加到暂存区，且有提示我们可以使用：`git rm --cached 文件名`,来取消添加到暂存区。

`git add`命令搭配通配符使用：
例：`git add *.txt`,会把所有.txt文件提交：
![[Pasted image 20260122170053.png]]
新建了三个.txt文件，一个sh文件，用上述命令提交，可见生效，且sh文件未被提交。

`git add`还能接收“文件夹”作为参数，表示**将某文件夹中的全部文件添加到暂存区：**
如`.`表示当前目录，`git add .`就表示将当前文件夹目录下的所有文件add：
![[Pasted image 20260122170534.png]]


#提交文件到本地仓库 ：
`git commit`: 该命令只会将暂存区中的文件提交到本地仓库。
![[Pasted image 20260121214016.png]]
创建新的gitFile2.txt (未添加到暂存区)，直接commit失败

`git commit -m "xxx"`: -m参数来指定提交时的信息，该信息会被记录到仓库中。
若不指定该参数则进入交互式界面通过vim来编辑提交信息。
vim的交互式界面：
![[Pasted image 20260121215151.png]]
**注意使用方法**：
接下来在 vim 里写提交信息并保存退出：
1. 按 `i` 进入输入模式
    
2. 输入提交信息，比如：`add second file`
    
3. 按 `Esc`
    
4. 输入 `:wq` 回车（保存并退出）
这里我在vim里输入的信息是：add 2nd File，结果如图所示：
![[Pasted image 20260121215309.png]]

有一个**易错点**：我执行了`git commit gitFile1.txt`这样一句命令,
出现报错：![[Pasted image 20260121214911.png]]
问题在于：执行了 `git commit gitFile1.txt`，Git 以为我要**打开编辑器写提交信息**，但它默认编辑器被设置成了 **vi**，而你这边的 vi 没装/打不开，所以报错了。

**另外**：这个写法本身也不对，**commit 不接文件名！** **文件是用 `git add` 选的**，commit 要写的是“提交说明”。

#查看提交记录： `git log`
`git log`查看：
![[Pasted image 20260122171638.png]]
每一次提交都有唯一编码，作者名称、邮箱，提交时间和注释信息。

查看简洁提交记录：
`git log --oneline`:
![[Pasted image 20260122171902.png]]

#回退版本 ：`git reset`:
`git reset`的三种模式：
![[Pasted image 20260122172111.png]]
√和×表示是否保存该区中的内容；mixed是默认参数。

新建一个提交了三次文件的仓库来演示回退：
![[Pasted image 20260122172806.png]]

复制三份文件分别演示三种模式的区别：
*（复制的每个文件都有着三次提交文件的记录），其实还有一个第一次提交是我手欠把父目录下所有文件commit了一下。这发生在我在父目录下创建了repo后，所以我刚创建的repo就因此有了一次初始的commit记录，后面又commit了3次文件，故有着四次log，所以后面复制的三个副本文件的commit也是有四次*
![[Pasted image 20260122173200.png]]

## 1.`git reset --soft 回退的目标版本id`：回退到上一个版本时，工作区和暂存区文件都保留
![[Pasted image 20260122174109.png]]
- 一开始指针指向第三次提交,id为74f835c。
- 指明回退到上一次版本，id为：7c3c767后。
- 回退成功，指针指向：7c3c767。
### `reset --soft` 不会改变“追踪清单”，只会移动 HEAD
`git reset --soft <commit>` 做的事是：
- **HEAD 回到 `<commit>`**
    
- **暂存区/工作区保持不动**

查看工作区：
![[Pasted image 20260122175314.png]]
第三次是被回退的，但是3.txt文件正常存在工作区中。

查看暂存区里被Git追踪的文件列表：*它反映的是 **index（暂存区）里记录的路径集合**（也就是 Git 认为纳入版本控制的文件清单）。*
`git ls-files`:
**这里有操作失误**：
![[Pasted image 20260122180831.png]]
>可见Git追踪的文件为空？
我把 `repo` 复制成 `repo-soft / repo-mixed / repo-hard` 用来练 `git reset`
    
- 复制后发现 `git ls-files` 为空，`git status` 表现异常
    
- 进一步检查：`ls -l .git/index`
显示：`No such file or directory`

**结论：副本仓库丢了 `.git/index`（暂存区文件）**  
所以 Git 没有“追踪清单”，`git ls-files` 就是空的。


## 正确创建文件副本：
```bash
cp -a repo repo-soft
cp -a repo repo-mixed
cp -a repo repo-hard
```
`-a` 会把隐藏文件一起复制（包括 `.git/index`）。
![[Pasted image 20260122181556.png]]

还有，仓库关系混乱了：
- 我在 `D:/learnGit` 下面创建了 `repo`，但 **`repo` 里没有 `.git`**，那它就不是独立仓库——它只是 **上级仓库 `D:/learnGit/.git` 管理的一个普通文件夹**。
- 我一直在用上级 `D:/learnGit/.git` 在做提交，所以 `repo`/`repo-soft` 其实只是上级仓库的子目录，不是独立仓库。
>现把learnGit取消掉仓库。
![[Pasted image 20260122182755.png]]

解决办法：
> 新建repo文件夹并初始化Git仓库，然后用`cp -a xxx`重新复制三份副本：
> ![[Pasted image 20260122184923.png]]
> 进入repo-soft副本文件夹可见三个文件都是正常被Git追踪的。

再测试回退到上一个版本：
![[Pasted image 20260122185257.png]]

`git reset --soft 3e625db`：
![[Pasted image 20260122185400.png]]

HEAD指针正常移动：
![[Pasted image 20260122185739.png]]

工作区一切正常：
![[Pasted image 20260122185438.png]]

暂存区一切正常：
第三次提交的3.txt也是被追踪的：
![[Pasted image 20260122185508.png]]

###  --soft参数不会删除工作区和暂存区的文件，但是版本回退到了第二次提交，此时3.txt相对来说是新的，add的，还未commit的文件，可以在此对其进行修改，然后重新commit。

## 2.`git reset --hard 回退的目标版本id`：回到上个版本时工作区和暂存区内容都清空
HEAD^表示上一条commit记录；
提交前：
![[Pasted image 20260125182524.png]]

执行：`git reset --hard HEAD^`:HEAD指针前移
![[Pasted image 20260125182553.png]]

查看暂存区文件：file3已消失
![[Pasted image 20260125182623.png]]、

查看工作区文件：file3也消失
![[Pasted image 20260125182649.png]]

## 3.`git reset --mixed（默认，可不写） 回退的目标版本id`：回到上个版本时，工作区保留，但暂存区不保留。
回退前：
![[Pasted image 20260125183125.png]]

执行：`git reset HEAD^`，因为默认就是mixed，这里没写。

查看git日志：
![[Pasted image 20260125183217.png]]

查看暂存区文件：
![[Pasted image 20260125183228.png]]

查看工作区文件：
![[Pasted image 20260125183238.png]]


## Git回溯：
- 查看操作历史记录：`git reflog`
- 找到误操作前的版本号：`git reset --hard  版本号`即可

*用HEAD指针来说明版本：
## HEAD：指向分支的最新提交节点（指针）
## HEAD^/HEAD~ ：最新提交版本的上一个版本
## HEAD~n :最新版本的前n个版本  


## Git 比较差异的命令：`git diff`
图解：
![[Pasted image 20260125214600.png]]

1.`git diff`默认比较的是工作区和暂存区中文件的差异：
以repo文件夹演示：
![[Pasted image 20260125184537.png]]
工作区有三个txt文件
![[Pasted image 20260125184550.png]]
也已经将这些文件add到暂存区中。

- 接下来对工作区中3.txt的内容作修改：
`vi 3.txt`:
将原本的内容：3，改为：G.E.M，完成后退出。
运行`git diff`:
![[Pasted image 20260125184802.png]]
index是Git根据文件内容生成的四十位的哈希值，后面的100644是访问权限；
>可见，由于我已将三个文件add到暂存区，又在工作区（本地）对3.txt作了修改，Git追踪的这两个区域间的差异：
红色部分"-"后是文件原来的内容，绿色“+”后是修改后的内容。

## 比较工作区和版本库：`git diff HEAD`
![[Pasted image 20260125205632.png]]

## 比较暂存区和版本库：`git diff --cached`
- 目前仅本地工作区变化，比较暂存区和版本区：
![[Pasted image 20260125205849.png]]
可见是无变化的。

- 进行add提交到暂存区：
![[Pasted image 20260125205951.png]]

- 再比较暂存区和版本库：
![[Pasted image 20260125210014.png]]
此时显示差别。

- commit后再对比工作区和版本库：无差别
![[Pasted image 20260125210130.png]]
- 对比暂存区和版本库：无差别
![[Pasted image 20260125210358.png]]
**则工作区、暂存区、版本库内容都一致**

## `git diff 版本1序号 版本2序号`：用于比较版本间的差异
*注意，是比较**从**“版本1” 到“版本2”**的过程中**，有哪些差异，**是有顺序的***
查看版本提交日志：
![[Pasted image 20260125210811.png]]

对比第二、第三次版本提交的差异：
`git diff 65e0450 3e625db`
![[Pasted image 20260125210959.png]]
*这里必须要说明，因为65e0450是新的版本，3e625db是老版本，这句命令意思是从新到旧进行比较，而唯一有变化的是3.txt的出现，但在这里是“删除”，是因为3.txt是3e625db后才有的，从“有”的版本走向“没有的版本”，只能是“删除”掉；这里如果把两个版本号互换：从老到新，就是正常的新增“new file mode”了。*

这段是 `git diff <提交A> <提交B>` 的输出，意思是：**从提交 `65e0450` 走到提交 `3e625db` 的变化里，文件 `3.txt` 被删除了**。
详细介绍：
- `diff --git a/3.txt b/3.txt`  
    表示对比的对象是同一个文件：旧版本路径叫 `a/3.txt`，新版本路径叫 `b/3.txt`（`a/`、`b/` 只是 diff 里的标记，不是你磁盘真实目录）。
    
- `deleted file mode 100644`  
    表示这个文件在新提交里**被删掉了**。`100644` 是普通文件权限（可读写/只读之类的标记），不用太纠结。
    
- `index 00750ed..0000000`  
    表示旧版本这个文件的内容哈希是 `00750ed`，新版本是 `0000000`（因为文件没了，所以是全 0）。
    
- `--- a/3.txt` 和 `+++ /dev/null`  
    `---` 是旧文件，`+++` 是新文件。`/dev/null` 表示“新文件不存在”，也就是删除。
    
- `@@ -1 +0,0 @@`  
    这是“改动范围”的标记：旧文件从第 1 行开始有 1 行；新文件从第 0 行开始有 0 行（因为文件被删了）。
    
- `-3`  
    这一行前面的 `-` 表示**从旧版本移除**的内容。也就是说 `3.txt` 原来只有一行内容是 `3`，现在这行没了，因为文件整体被删。


## 在提交的两个版本间，只比较特定文件的差异：
`git diff 版本号1 版本号2 文件名`：
`git diff HEAD~2 HEAD 3.txt`
![[Pasted image 20260125214349.png]]

## 比较分支间差异：
`git diff 分支名 分支名`


## 删除版本库文件：
1.本地删除后，commit更新：
- 先本地删除文件1.txt：
![[Pasted image 20260125215130.png]]
- 查看仓库状态：可见其监测到1.txt被删除：
![[Pasted image 20260125215204.png]]
**这里注意**：提示我们要git add更新即将要被提交的文件。
>相当于我们现在删除的1.txt在暂存区里还是有副本准备去提交的，此时我们需要add一下，就会把没有1.txt的工作区更新到暂存区，作用相当于：更新1.txt“被删除”这一消息，接下来commit到仓库中的文件自然也就不会有1.txt了。
![[Pasted image 20260125215525.png]]
>可见，git add 后再查看仓库状态，删除1.txt这一信息已被仓库"确认"。

2.`git rm 文件名`：把文件从工作区和暂存区中同时删除：
`git rm 2.txt`,将2.txt删除:
![[Pasted image 20260125215916.png]]

3.`git rm --cached 文件名`：把文件从暂存区中删除，但工作区中保留,可将文件修改后重新add，再commit。
`git rm --cached 3.txt`:
![[Pasted image 20260125220253.png]]


总结：
![[Pasted image 20260125220048.png]]


## .gitignore文件：在其中配置不提交到版本库中的文件，以保证仓库体积不会太大、也更干净。
应该忽略的文件：
![[Pasted image 20260125223306.png]]
.gitignore文件里存的是要忽略的文件的文件名。

- 新建一个日志文件：
![[Pasted image 20260125224302.png]]
正常情况下Git会将其视为要提交到仓库的文件，而提示我们将其添加到暂存区：
![[Pasted image 20260125224322.png]]****

### 把access.log（新建的日志文件）的文件名，配置进.gitignore文件中，含义是告诉Git忽略掉这个文件，它不用提交到仓库，所以不用管它！
![[Pasted image 20260125224731.png]]

此时查看仓库状态，Git已不再视日志文件为应该追踪的文件（但新增了一个.gitignore文件，这个文件会被Git追踪，因为要时刻知道哪些文件不需提交到仓库）：
![[Pasted image 20260125224812.png]]

把进行的修改都添加到暂存区：
![[Pasted image 20260125225242.png]]

进行提交到仓库：
![[Pasted image 20260125225411.png]]

查看暂存区中被Git追踪的文件：
![[Pasted image 20260125225501.png]]
可见access.log被配置到.gitignore文件中后，提交时没有将其提交到仓库中，暂存区中也没有对其进行追踪。

- 可配置通配符对所有日志文件进行忽略：`*.log`
![[Pasted image 20260125230100.png]]
![[Pasted image 20260125230044.png]]
>此时直接新增一个日志文件看Git会不会“管它”：
![[Pasted image 20260125230316.png]]

此时看追踪情况，可见Git只注意到.gitignore的变化（因为还没commit提交保存），而自动忽略了新增的日志文件。
![[Pasted image 20260125230423.png]]

**.gitignore文件只对未被提交到仓库的文件生效**，如果文件已经被提交到仓库中，该文件就被视作正常文件进行跟踪，不论是修改文件内容还是git diff都是有效的。

## .gitignore 可以配置忽略文件夹：
*空文件夹不会被Git纳入到版本控制中，文件夹中有内容才会*。
此时创建的temp文件是空的，Git不予理会：
![[Pasted image 20260126002555.png]]

文件夹下有内容时，Git纳入版本控制，提示我们git add来追踪：
![[Pasted image 20260126002651.png]]

对文件夹进行忽略：在.gitignore文件夹中写：“文件夹/”这样的格式，如temp/。
![[Pasted image 20260126002841.png]]
配置好再`git commit -am "略"`修改,
*`commit -am "msg"`:*
- `-a`：**把所有“已被追踪（tracked）”的文件**里发生的修改/删除，自动 `git add` 到暂存区（相当于对这些文件执行 add/rm）
    
- 然后 `commit`：把这些已暂存的改动提交掉
    
- `-m`：顺带写提交信息
！！
1.不会包含新文件，新文件必须先 `git add 新文件`，被追踪后，之后才可以用 `-am` 方便提交后续修改。
2.会包含 **删除**（如果你把一个 tracked 文件删了，`-a` 会把这个删除也 stage 并提交）。

## .gitignore文件匹配规则：
- 从上到下逐行匹配，每一行表示一个忽略模式
*注：不是Blob，是Glob模式*
![[Pasted image 20260126004337.png]]

示例：
![[Pasted image 20260126004659.png]]
![[Pasted image 20260126004708.png]]
