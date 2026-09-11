>每个分支都有一个HEAD指针指向最新提交，执行rebase时，Git会找到当前分支和目标分支的共同祖先节点（提交记录），作为”分叉点“，然后把当前分支自从”祖先节点“后一直到最新的提交记录都移动到目标分支的最新提交后面。（类似嫁接移植）

回顾merge进行合并
![[Pasted image 20260126230756.png]]
结果：main作为合并的目标分支，合并后在末尾会新增一次提交，然后两条分支像溪流一样“汇集“在一起。

而变基（Rebase）：`git rebase xxx`:现在将xxx视作”基底“，把当前分支上的记录变基迁移到指定的xxx分支，xxx就是目标。

如在dev分支执行rebase：
```bash
git switch main
git rebase dev
```
![[Pasted image 20260126231031.png]]
![[Pasted image 20260126231102.png]]
即把dev分支上版本记录的”基底“更改为”另一条分支“main，从而dev分支上的版本记录都变成”基于main分支的版本记录“，从而发生迁移。

而在main分支上执行rebase：
```bash
git switch main
git rebase dev
```
![[Pasted image 20260126231517.png]]
![[Pasted image 20260126231537.png]]

## merge和rebase的对比：
1.merge:
优点：不破坏原分支的提交历史，方便回溯、查看
缺点：产生额外的提交节点，分支图较复杂
![[Pasted image 20260126234220.png]]

2.rebase：
优点：不新增额外的提交记录，形成线性的提交历史，较直观干净
缺点：会改变提交历史
（这是相对于merge而言，因为merge合并后仍会保存每条分支的提交历史，可以按需对不同的分支执行不同的回退策略），rebase则会改变当前分支branch out的节点，应避免在共享分支使用。
![[Pasted image 20260126235058.png]]
![[Pasted image 20260126235115.png]]
***************
在dev分支上执行rebase：
![[Pasted image 20260126235154.png]]
***************
结果：
![[Pasted image 20260126235210.png]]

---

## rebase / reflog / reset 重写历史（博客笔记）

> 来源：http://120.77.152.123:8088/posts/git%e7%9a%84%e4%b8%80%e4%ba%9b%e4%bd%bf%e7%94%a8 ｜ 原发布日期：2026-08-11

#### 场景：前两次commit的msg写的有点问题，想重写（未push）

##### 安全，在本地git仓库修改：

若想修改最近一次，`git commit --amend`,在弹出文本编辑器重新编辑commit msg。

前几次

- rebase -i ：

交互式变基，先`git rebase -i HEAD~3`看commit范围，在文本编辑器中将需要改动的commit对应的前缀由`pick`改为`r/reword`

如：

```
reword 1234567 修复了xxbug
pick 89abcde 添加了加载动画
r f1a2b3c 修改了xxx的缩进
```

后续逐个修改具体的 commit 信息： 刚才保存退出后，Git 会自动停在你标记了 reword 的每一个 commit 处，并再次弹出一个编辑器，让你修改那条特定的 commit 信息。

#### Rebase变基，本地安全，远端可能导致严重冲突：

##### 为什么需要变基：

在 Git 中，任何一个 commit 生成后，它的内容（包括提交信息）和它前一个 commit 的状态是紧密绑定在一起的。 如果你想修改倒数第 3 个 commit 的信息，Git 是不能在原地直接改文字的。

Git需要：

1. 回到倒数第三个commmit前一个commit
2. 重新根据需要修改的信息，生成倒数第三个commit
3. 把倒数第 2 个、倒数第 1 个 commit，像搭积木一样，重新“变基”（贴回） 到新的倒数第 3 个 commit 上面。

修改以往历史本质： 将需要修改的commit重建，然后将后续的commit重新拼接到这个新commit后方（把后面的 commit 重新应用在一棵修改过的树根上）

###### 危险情况：

如果这几个 commit 已经`推送到了远端`（哪怕你只是刚刚 push 上去），这时候你再去 `rebase 修改历史`，就会导致你本地的历史和远端服务器的历史`“分叉”`。此时你只能`强推（git push -f）`，这就非常危险了，极其容易`把同事刚好提交的代码给覆盖掉。`

###### Git的兜底机制，防修改出错：

- 正在操作时：`git rebase -i` 时，修改错了commit，或者将msg改乱了，只需不报错退出编辑器，或在命令行里输入 `git rebase --abort`，这次修改就会立刻中止，无事发生。
- 同样，如果**已经** 操作完成了，也可以`放弃掉rebase`：

1.用`git reflog`查看最近执行的git命令 2. 然后找到`错误rebase前`的commit 3. 直接`reset`掉`HEAD`即可，如`git reset --hard HEAD@{4}/该commit的hash值`，就直接重置了本地git仓库的`HEAD指针`，重回到某个历史状态，实现了放弃错误rebase。
