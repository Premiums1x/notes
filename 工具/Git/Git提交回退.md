---
title: "Git提交回退"
date: 2026-07-28
section: "Internship"
source: http://120.77.152.123:8088/posts/Git%e5%ae%9e%e6%93%8d
tags:
  - "Internship"
  - "博客迁移"
---
今天提交代码后发现前一次的commit message写错了，解决： 因为是本地的commit还没push到远端仓库，所以对于更改提交历史友好。

- 先撤销最新提交：`git reset --soft HEAD~1`
- 然后最新那次提交的代码会被`stage`暂存，把文件从`stage`里取出到未暂存状态，否则如果改旧的commit信息，会把两部分都提交上去：`git restore --staged .`
- 修改commit信息，用`--amend`选项：`git commit --amend -m "xxx"`,改好之后在把最新的改动`git add .`,`git commit -m "xx"`

之前自己写东西习惯了做点什么commit完就push，实际业务开发协作里还是要注意千万不能动不动就push，切实体会到只要没push到远端就一切好说。
