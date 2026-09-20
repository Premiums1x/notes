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

## git restore：理解“恢复工作区”和“取消暂存”

理解 `git restore` 前，先固定 Git 的三个区域：

- **HEAD**：当前分支最近一次已经完成的 commit。
- **暂存区（Index / Staging Area）**：下一次 commit 准备提交的内容。
- **工作区（Working Tree）**：磁盘上现在正在编辑的文件。

正常的数据流是：

```text
工作区 -- git add --> 暂存区 -- git commit --> HEAD
```

而 `restore` 可以理解为沿相反方向“恢复”：

```text
HEAD -- git restore --staged --> 暂存区
暂存区 -- git restore --------> 工作区
```

### 1. `git restore 文件名`：恢复工作区

```bash
git restore a.js
```

它操作的是**工作区**，默认使用**暂存区中的版本**覆盖工作区：

```text
暂存区 → 工作区
```

所以如果文件还没有 `git add`：

```text
HEAD       A
暂存区     A
工作区     B
```

执行：

```bash
git restore a.js
```

结果：

```text
HEAD       A
暂存区     A
工作区     A
```

也就是：**未暂存的修改 B 被丢弃，文件恢复到暂存区中的内容。**

> 注意：这通常意味着未提交的工作区修改会被直接丢弃，执行前应先确认这些修改确实不要了。

### 2. 已经 add 后，不加 `--staged` 会怎样？

假设：

```bash
# 修改文件
git add a.js
```

此时：

```text
HEAD       A
暂存区     B
工作区     B
```

再执行：

```bash
git restore a.js
```

因为普通 `restore` 做的是：

```text
暂存区 → 工作区
```

而此时暂存区和工作区本来都是 B，所以**看起来基本什么都没发生**。

文件依然处于 staged 状态，`git add` 并没有被撤销。

#### 更需要注意的情况：add 后又继续修改

例如：

```text
HEAD       A
暂存区     B        # 第一次修改后执行了 git add
工作区     C        # add 之后又继续修改
```

此时执行：

```bash
git restore a.js
```

会得到：

```text
HEAD       A
暂存区     B
工作区     B
```

也就是说，**add 之后继续产生的工作区修改 C 会被丢掉**，但已经暂存的 B 仍然保留。

因此：

> 文件已经 add，并不代表普通 `git restore 文件名` 会自动“取消暂存”。

### 3. `git restore --staged 文件名`：取消暂存

如果目的是撤销 `git add`，应该使用：

```bash
git restore --staged a.js
```

它操作的是**暂存区**，默认使用 HEAD 中的版本恢复暂存区：

```text
HEAD → 暂存区
```

例如 add 后：

```text
HEAD       A
暂存区     B
工作区     B
```

执行：

```bash
git restore --staged a.js
```

结果：

```text
HEAD       A
暂存区     A
工作区     B
```

因此：

- 暂存状态被取消；
- 工作区里的代码修改 B **还在**；
- 只是这些修改不再属于“下一次准备 commit 的内容”。

这就是“取消暂存但保留代码”。

### 4. 为什么 `--staged` 是 HEAD → 暂存区？明明还没 commit

关键是：

> **HEAD 不是“准备提交的内容”，HEAD 指向的是当前分支最近一次已经存在的 commit。**

例如初始状态：

```text
HEAD       A
暂存区     A
工作区     A
```

修改文件后：

```text
HEAD       A
暂存区     A
工作区     B
```

执行：

```bash
git add a.js
```

得到：

```text
HEAD       A
暂存区     B
工作区     B
```

注意：这时只是 add，**还没有产生新的 commit**，因此 HEAD 仍然是 A。

现在如果想“撤销 add”，Git 就需要让暂存区恢复到 add 之前、也就是最近一次 commit 的状态：

```text
HEAD A → 暂存区
```

于是：

```bash
git restore --staged a.js
```

执行后：

```text
HEAD       A
暂存区     A
工作区     B
```

所以 `--staged` 会引用 HEAD，恰恰是因为**当前修改还没有 commit**：HEAD 仍然保存着这次修改之前的已提交版本，可以作为恢复暂存区的参照。

### 5. 两条命令最简单的记法

```bash
git restore a.js
```

记成：

```text
暂存区 → 工作区
```

即：

> “让工作区重新和暂存区一样。”

而：

```bash
git restore --staged a.js
```

记成：

```text
HEAD → 暂存区
```

即：

> “让暂存区重新和 HEAD 一样。”

完整关系：

```text
正常前进：

工作区 -- git add --> 暂存区 -- git commit --> HEAD


恢复：

工作区 <-- git restore -------- 暂存区
             暂存区 <-- git restore --staged -- HEAD
```

### 6. 已经 add，但现在连修改也不要了

如果一个修改已经进入暂存区：

```text
HEAD       A
暂存区     B
工作区     B
```

可以先取消暂存：

```bash
git restore --staged a.js
```

变成：

```text
HEAD       A
暂存区     A
工作区     B
```

然后再丢弃工作区修改：

```bash
git restore a.js
```

最终：

```text
HEAD       A
暂存区     A
工作区     A
```

也就是：

```text
先：HEAD → 暂存区
再：暂存区 → 工作区
```

### 7. 常用命令速记

| 命令 | 作用 |
| --- | --- |
| `git restore file` | 恢复工作区文件，丢弃未暂存修改 |
| `git restore .` | 恢复当前目录所有工作区修改 |
| `git restore --staged file` | 取消单个文件的暂存，但保留工作区修改 |
| `git restore --staged .` | 取消所有暂存，但保留工作区修改 |

核心不要记成“文件 add 以后，restore 的作用就自动变了”，而应该记：

> **有没有 `--staged` 决定 restore 操作哪个区域。**

- 不带 `--staged`：恢复**工作区**。
- 带 `--staged`：恢复**暂存区**。
