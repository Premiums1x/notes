---
title: "Git核心概念"
date: 2026-09-01
section: "Debug"
source: http://120.77.152.123:8088/posts/git%e7%9b%b8%e5%85%b3
tags:
  - "Debug"
  - "博客迁移"
---
分支指针、HEAD，远端和本地Git仓库

git fetch与git pull区别

git merge的两个parent节点，merge只保存commit history，具体代码要看相对base的修改

git reset: 1. --hard 2. -- mixed(默认) 3. --soft，三者的区别：

```
模式   HEAD / 分支指针	暂存区 Index	工作区 Working Tree
--soft	回到目标 commit	不动	不动
--mixed	回到目标 commit	回到目标 commit	不动
--hard	回到目标 commit	回到目标 commit	回到目标 commit
```

```
soft
只退 commit
→ “我只是后悔 commit 了，暂存好的东西别动”

mixed
退 commit + 退暂存区
→ “我要重新挑一次哪些东西应该 add”

hard
commit + 暂存区 + 工作区一起退
→ “这些修改我全都不要了”
```

git ls-files 文件名：看文件有无被纳入版本管理

git rebase变基改变commit历史， 配合-i选项可以启动交互式变基，适用于需要改变的commit是历史commit,而非当前最新，将需要修改的commit前方修改为edit，然后退出，改动完成后 git rebase --continue ； git revert：生成一个新commit与前一个commit操作相反

git reflog 看HEAD指针变化日志

git diff 看差异，不加--参数就是看工作区与xx的差别 如：git diff HEAD origin/main -- .gitigonre，还能看特定文件的差异

git的ignore文件是忽略未被追踪文件，若已纳入版本管理，需 git rm --cached 文件名，如果删的是目录还要加--r，递归删除。

git amend:“修补”，把当前暂存区内的文件直接合并覆盖到最新的一次commit中，不产生新的commit； 一般配合--no-edit使用，表示沿用最新一次的commit msg，不用弹窗文本编辑器来进行确认。
