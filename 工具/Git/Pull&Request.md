---
title: "Pull&Request"
date: 2026-07-16
section: "Github"
source: http://120.77.152.123:8088/posts/PR
tags:
  - "Github"
  - "博客迁移"
---
Pull&Request 为了代码评审

PR的两个阶段：Approve和Merge

需要先Approve然后Create PR

如果不能Auto Merge就说明有文件存在冲突：

需要先解决文件冲突再重新PR： 刚刚两个worktree：codex/github和codex/lancer

前者签出较早，和后者的代码出现冲突： 先把main merge到codex/github，解决冲突后再在codex/github分支 PR
