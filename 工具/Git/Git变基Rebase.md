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
