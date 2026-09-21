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


---

## 实战：代码误推正式分支后的修复（cherry-pick + revert + stash）

> 场景：原本应该提交到功能分支的代码，不小心 commit 并 push 到了正式/发布分支；与此同时，本地又已经开始开发新的需求，工作区还有未提交的修改。

这个场景实际上同时涉及三个问题：

1. **当前正在开发的新需求不能丢**：先用 `git stash` 临时保存工作区。
2. **误提交的代码本身是有用的，只是待错了分支**：用 `git cherry-pick` 把对应 commit 拿到正确的功能分支。
3. **正式分支已经被推到远端，不能随便改写公共历史**：用 `git revert` 生成一个反向 commit，把错误修改撤回来。

整体思路：

```text
正在开发的新需求
        │
        │ git stash
        ▼
   临时保存起来
        │
        │
        ├───────────────┐
        │               │
        ▼               ▼
正确功能分支          正式分支
cherry-pick           revert
拿到错误 commit       撤销错误 commit
        │               │
        ▼               ▼
push 功能分支         push 正式分支
        │
        └───────┐
                ▼
          git stash pop
          恢复新需求开发
```

### 1. 先保护当前正在开发的新需求：git stash

如果工作区还有新需求代码，此时直接切分支，Git 可能拒绝切换，或者修改会跟着一起带到别的分支。

先看状态：

```bash
git status
```

然后把当前工作保存起来：

```bash
git stash push -u -m "wip: 当前新需求"
```

其中：

- `stash`：把当前未提交的修改临时存起来。
- `push`：明确表示“创建一条 stash”；现在更推荐写完整形式。
- `-u` / `--include-untracked`：除了已被 Git 跟踪的文件，还把 **untracked 文件** 一起保存。
- `-m`：给这条 stash 添加说明，避免以后只看到一堆难以区分的 `WIP on ...`。
- `-u` **不会包含被 .gitignore 忽略的文件**；如果连 ignored 文件都要保存，可以了解 `-a / --all`，但日常要谨慎使用。

也可以省略 `push`：

```bash
git stash -u -m "wip: 当前新需求"
```

但完整写法更容易理解：

```bash
git stash push -u -m "wip: 当前新需求"
```

#### 查看所有 stash

```bash
git stash list
```

例如：

```text
stash@{0}: On feature/new-demand: wip: 当前新需求
stash@{1}: On develop: wip: yesterday work
```

`stash@{0}` 通常是最近一次 stash。

想看看某一条 stash 改了什么：

```bash
git stash show stash@{0}
```

查看详细 diff：

```bash
git stash show -p stash@{0}
```

---

### 2. 找到推错分支的 commit

先查看提交记录：

```bash
git log --oneline
```

例如：

```text
a1b2c3d feat: 新增订单筛选功能
9f8e7d6 fix: 修复列表样式
...
```

假设：

```text
a1b2c3d
```

这条 commit 本来应该属于：

```text
feature/order-filter
```

但误提交到了：

```text
release
```

此时不要把这条 commit 当成“垃圾提交”。

**代码本身是正确的，只是 commit 所在的分支错了。**

所以需要做两件独立的事：

```text
feature/order-filter  ← 把 commit 拿过来
release               ← 把 commit 的效果撤销
```

对应：

```text
cherry-pick = 拿 commit
revert      = 反做 commit
```

---

## git cherry-pick：把某个 commit 跨分支拿过来

### 1. 基本作用

`git cherry-pick` 可以理解成：

> 把其他分支上的某一个 commit 的修改取出来，在当前分支重新生成一个新的 commit。

例如错误 commit：

```text
release
A --- B --- C
          ↑
       a1b2c3d
```

切换到功能分支：

```bash
git switch feature/order-filter
```

然后：

```bash
git cherry-pick a1b2c3d
```

结果类似：

```text
release
A --- B --- C
          ↑
       a1b2c3d


feature/order-filter
A --- D
      ↑
   cherry-pick 后的新 commit
```

注意：

> cherry-pick 复制的是 **commit 带来的修改**，而不是直接把原 commit 对象搬过来。

因此新生成的 commit 通常会有一个新的 SHA。

### 2. 推荐操作流程

```bash
# 切到真正应该放代码的功能分支
git switch feature/order-filter

# 确认当前分支
git branch --show-current

# 把错误分支上的 commit 拿过来
git cherry-pick a1b2c3d

# 检查
git status
git log --oneline -5

# 推到远端功能分支
git push origin feature/order-filter
```

### 3. 一次 cherry-pick 多个 commit

如果是几个离散 commit：

```bash
git cherry-pick <commit1> <commit2> <commit3>
```

如果是一段连续提交，也可以使用 commit range，但在不熟悉范围语法前，更推荐先：

```bash
git log --oneline
```

确认清楚后逐个 cherry-pick，降低误操作概率。

### 4. cherry-pick 出现冲突

如果目标分支和原 commit 修改了相同位置，可能出现冲突。

此时：

```bash
git status
```

手动解决冲突后：

```bash
git add .
git cherry-pick --continue
```

如果发现不应该 cherry-pick，想整个取消：

```bash
git cherry-pick --abort
```

记忆：

```text
解决完冲突 → --continue
整个操作不要了 → --abort
```

---

## git revert：安全撤销已经推到远端的错误 commit

### 1. revert 的本质

假设正式分支已经有：

```text
A --- B --- C
          ↑
      错误 commit
```

执行：

```bash
git revert C
```

Git **不会删除 C**，而是新增一个提交 D：

```text
A --- B --- C --- D
          ↑       ↑
       错误提交   反向提交
```

其中：

```text
C：+100 行代码
D：-100 行对应代码
```

从最终文件内容看，C 的效果被抵消了。

但历史仍然完整：

```text
C 曾经被提交过
D 后来明确撤销了 C
```

### 2. 为什么正式/共享分支应该优先用 revert

如果错误 commit **已经 push 到远端共享分支**，例如：

```text
main
master
release
develop（多人共享）
```

一般不要轻易使用：

```bash
git reset --hard <old-commit>
git push --force
```

因为这相当于：

> 改写远端已经存在的提交历史。

其他同事可能已经基于这段历史继续开发，强推后容易导致大家的本地历史发生分叉甚至丢提交。

而 `git revert`：

- 不删除历史；
- 不移动已经公开的历史；
- 只是新增一个“撤销提交”；
- 普通 `git push` 即可；
- 更适合共享分支和正式分支。

所以可以简单记：

```text
还没推远端、只是自己本地玩 → reset 可以考虑
已经推到共享远端          → 优先 revert
```

这不是绝对规则，但作为团队开发的默认习惯非常实用。

### 3. 正式分支修复流程

假设错误 commit 是：

```text
a1b2c3d
```

切回正式分支：

```bash
git switch release
```

确认分支：

```bash
git branch --show-current
```

为了避免本地正式分支太旧，可以先同步远端：

```bash
git pull --ff-only origin release
```

然后撤销错误提交：

```bash
git revert a1b2c3d
```

Git 会创建一个新的 revert commit。

检查：

```bash
git status
git log --oneline -5
```

最后：

```bash
git push origin release
```

### 4. 多个错误 commit 怎么办

如果误推了多个互相关联的 commit，最容易理解的做法是：

> 从最新的错误 commit 开始，一个个往前 revert。

例如：

```text
A --- B --- C --- D
          ↑     ↑
        错误1  错误2
```

可以：

```bash
git revert D
git revert C
```

这样比较符合“后做的操作先撤销”的思路，也更容易处理前后依赖。

如果发生冲突：

```bash
git status
```

解决冲突并暂存：

```bash
git add .
git revert --continue
```

如果想放弃当前这次 revert：

```bash
git revert --abort
```

---

## cherry-pick 和 revert 的区别

| 命令 | 核心目的 | 是否生成新 commit | 常见场景 |
| --- | --- | --- | --- |
| `git cherry-pick <sha>` | 把某个 commit 的修改拿到当前分支 | 是 | 代码提交到了错误分支，但修改本身要保留 |
| `git revert <sha>` | 生成一个与目标 commit 效果相反的提交 | 是 | 已经 push 到共享分支，需要安全撤销 |
| `git reset` | 移动当前分支指针 | 不一定 | 本地整理历史、撤销尚未共享的提交 |

这次场景正好体现了二者可以配合：

```text
错误 commit
   │
   ├── cherry-pick → 正确功能分支
   │
   └── revert      → 正式分支撤销
```

也就是说：

> **cherry-pick 解决“这份代码应该去哪”，revert 解决“这份代码不应该留在哪”。**

---

## git stash：临时保存“做到一半”的工作

`git stash` 最适合这种情况：

> 新需求写到一半，突然需要切分支处理线上问题、合并冲突或修复错误提交，但当前代码还不值得 commit。

它相当于：

```text
当前工作区
   │
   │ git stash
   ▼
Git 临时储藏区
   │
   │ git stash pop
   ▼
恢复到工作区
```

### 1. 常用保存命令

只保存已跟踪文件的修改：

```bash
git stash push -m "wip: xxx"
```

连 untracked 新文件一起保存：

```bash
git stash push -u -m "wip: xxx"
```

本次场景中，更适合：

```bash
git stash push -u -m "wip: 新需求开发中，临时处理错误分支"
```

因为新需求经常会创建新的组件、页面或模块，这些文件还没有 `git add`，属于 untracked；如果不带 `-u`，它们仍然会留在工作区。

### 2. 查看 stash

```bash
git stash list
```

查看最近一条：

```bash
git stash show -p stash@{0}
```

### 3. 恢复 stash：pop

```bash
git stash pop
```

默认恢复最近一条，即：

```text
stash@{0}
```

也可以明确指定：

```bash
git stash pop stash@{0}
```

`pop` 的逻辑可以理解成：

```text
应用 stash
+
成功后从 stash list 中删除它
```

所以：

```bash
git stash pop
```

≈

```bash
git stash apply stash@{0}
git stash drop stash@{0}
```

但有一个重要细节：

> 如果 `stash pop` 恢复时发生冲突，Git 通常不会直接把这条 stash 删除，避免修改丢失。

发生冲突时先：

```bash
git status
```

再正常解决冲突。

### 4. pop 和 apply 的区别

```bash
git stash pop
```

恢复后，成功时会删除 stash。

而：

```bash
git stash apply stash@{0}
```

只恢复，stash 记录仍然保留。

因此如果是一份比较重要的临时修改、担心恢复过程有问题，可以先：

```bash
git stash apply stash@{0}
```

确认没问题后再：

```bash
git stash drop stash@{0}
```

### 5. 删除 stash

删除指定 stash：

```bash
git stash drop stash@{0}
```

清空全部 stash：

```bash
git stash clear
```

`clear` 风险比较高，执行前最好：

```bash
git stash list
```

确认没有重要内容。

---

## 本次事故的完整推荐操作顺序

假设：

- 正式分支：`release`
- 正确功能分支：`feature/order-filter`
- 错误 commit：`a1b2c3d`
- 当前工作区还有一份新需求在开发

### Step 1：保存当前新需求

```bash
git status
git stash push -u -m "wip: 新需求，临时处理错误提交"
git stash list
```

### Step 2：把错误 commit 拿到正确功能分支

```bash
git switch feature/order-filter
git cherry-pick a1b2c3d
git push origin feature/order-filter
```

如果冲突：

```bash
git status

# 手动解决冲突后
git add .
git cherry-pick --continue
```

### Step 3：正式分支撤销错误 commit

```bash
git switch release
git pull --ff-only origin release
git revert a1b2c3d
git push origin release
```

### Step 4：回到开发分支恢复新需求

先切回原来开发新需求的分支：

```bash
git switch feature/new-demand
```

确认 stash：

```bash
git stash list
```

然后：

```bash
git stash pop stash@{0}
```

最后：

```bash
git status
```

确认新需求代码已经恢复。

---

## 最重要的心智模型

### commit 是“修改记录”，分支只是指向 commit 的指针

这次之所以可以通过 `cherry-pick` 修复，是因为 Git 中的 commit 本身记录的是一次修改。

所以：

```text
“提交错分支”
```

不等于：

```text
“这份代码废了”
```

而是：

```text
这个 commit 出现在了错误的提交历史上
```

可以把它的修改重新应用到正确分支。

### 三个命令分别解决三个不同问题

```text
git stash
↓
我现在手里的未提交代码怎么办？

git cherry-pick
↓
已经存在的某个 commit，我想把它放到另一个分支怎么办？

git revert
↓
已经推到共享分支的错误 commit，我怎么安全撤销？
```

一句话记忆：

> **stash 管“未提交的工作”，cherry-pick 管“跨分支搬提交”，revert 管“安全撤销已共享的提交”。**
