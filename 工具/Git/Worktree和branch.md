---
title: "Worktree和branch"
date: 2026-08-25
section: "Github"
source: http://120.77.152.123:8088/posts/%e5%8c%ba%e5%88%86%e5%b7%a5%e4%bd%9c%e6%a0%91%e4%b8%8e%e5%88%86%e6%94%af
tags:
  - "Github"
  - "博客迁移"
---
`branch` 分支：

正常一条分支只对应一份代码目录，如果切换分支，当前整个代码目录就会被替换。

`分支是“另一条代码版本线”，worktree 是“把这条版本线同时展开到另一个文件夹”。`

`worktree` 工作树：

通过worktree可以创建一份独立的且绑定一条分支的代码目录（内容完整且配置独立），这份代码目录可以与当前代表每条git分支的代码目录共存。从而可以在不干扰已有git分支代码的情况下，启动多个代码版本的服务进行开发/测试....而不冲突。且便于用一版开发、测试闭环后的完整的代码发起pr与原来的分支进行合并。

使用示例：

```
git worktree add `
  .worktrees/tenant-invites-agent-ux `
  codex/tenant-invites-agent-ux
```
