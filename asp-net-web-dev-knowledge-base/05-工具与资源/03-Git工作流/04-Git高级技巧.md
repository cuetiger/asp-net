# Git 高级技巧

掌握 Git 基础命令后，高级技巧能够帮助你更高效地处理复杂场景、解决棘手问题、优化工作流程。本文深入介绍 Rebase、Cherry-pick、Bisect、Reflog 等高级功能，以及常见问题的解决方案。

## 一、Rebase vs Merge 深度对比

### 1.1 核心区别图解

```
Merge（合并）:
                    A --- B --- C  (feature)
                   /             \
main: X --- Y --- M --------------- D (merge commit)

特点：
- 创建合并提交 M，保留完整分支历史
- 非线性历史，有分叉和合并节点
- 时间线真实反映开发过程
- 安全性高，不易出错


Rebase（变基）:
main: X --- Y --- A' -- B' -- C' -- D (feature)

特点：
- 线性历史，无合并节点
- 将 feature 的提交"移动"到 main 最新位置
- 重写提交历史（新 hash）
- 历史更整洁但改变了时间线
```

### 1.2 详细对比表

| 维度 | Merge | Rebase |
|------|-------|--------|
| **历史形状** | 有分叉合并 | 线性整洁 |
| **提交 Hash** | 原始保留 | 全部改变 |
| **安全性** | 高（不重写历史） | 中等（重写历史） |
| **冲突处理** | 只需一次 | 每个提交可能都要处理 |
| **调试追溯** | 容易（保留上下文） | 较难（丢失上下文） |
| **适用范围** | 公共分支/任何情况 | 本地/个人分支 |
| **回滚难度** | 简单（revert 合并提交） | 复杂（需要找到原始提交） |

### 1.3 黄金法则

```
Git Rebase 黄金法则：

⚠️ 永远不要 rebase 已经推送到公共仓库的提交！

原因：
1. 其他协作者可能基于你的原始提交工作
2. Rebase 会改变提交的 hash
3. 协作者拉取时会产生大量冲突
4. 可能导致他人丢失工作

安全使用 Rebase 的场景：

✅ 允许的场景：
├── 清理本地未推送的提交
├── 将个人 feature 分支更新到最新 main
├── 整合本地提交（squash/reword）
├── 在 PR 合并前整理提交历史

❌ 禁止的场景：
├── Rebase 已推送到 main 的提交
├── Rebase 其他人正在工作的公共分支
├── 强制推送（push -f）到共享分支
└── 在团队不了解的情况下 rebase 共享代码
```

### 1.4 选择决策树

```
何时用 Merge vs Rebase？

开始
  │
  ▼
提交是否已推送？
  │
  ├── 是 ──► 使用 Merge
  │         （或 revert）
  │
  └── 否 ──► 是否是个人分支？
              │
              ├── 是 ──► 使用 Rebase ✅
              │           （保持历史整洁）
              │
              └── 否 ──► 团队约定是什么？
                          │
                          ├── 允许 Rebase ► Rebase
                          └── 不确定     ► Merge（更安全）
```

---

## 二、Interactive Rebase（交互式变基）

### 2.1 基本概念

Interactive Rebase 允许你在变基过程中修改提交历史，是整理提交记录的强大工具。

```bash
# 修改最近 N 个提交
git rebase -i HEAD~N

# 修改从某个提交之后的所有提交
git rebase -i <commit-hash>

# 示例：修改最近 3 个提交
git rebase -i HEAD~3
```

### 2.2 六种操作详解

执行 `git rebase -i HEAD~3` 后会打开编辑器显示类似内容：

```
pick f7f3f6d feat: add user login functionality
pick 310154e fix: resolve authentication bug
pick a5f4a0d docs: update README with new features

# Rebase 1234abcd..a5f4a0d onto 1234abcd
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C | -c] <commit> [# <oneline>]
# .       create a merge commit using the original merge message
# .       or the oneline if -c is specified
```

#### 操作一：pick（选取）

```bash
# 默认操作，保留提交不变
pick f7f3f6d feat: add user login functionality
```

#### 操作二：reword（修改提交信息）

```bash
# 修改提交信息（不改变代码）
r f7f3f6d feat: add user login functionality
# 执行后会弹出编辑器让你修改信息

# 适用场景：
# - 修正拼写错误
# - 使描述更加清晰
# - 统一提交格式规范
```

**实际示例**：

```
# 修改前：
feat: adds user login stuff

# 修改后（reword 后）：
feat(auth): implement JWT-based user authentication flow

- Add login endpoint POST /api/auth/login
- Generate and return access token
- Implement token refresh mechanism
```

#### 操作三：edit（编辑提交）

```bash
# 停在该提交，允许你修改代码
e 310154e fix: resolve authentication bug

# 流程：
# 1. Rebase 会停在这个提交
# 2. 你可以修改文件
# 3. git add 修改的文件
# 4. git commit --amend（或 git rebase --continue）
# 5. 继续 rebase 过程

# 适用场景：
# - 发现某次提交遗漏了文件
# - 需要拆分一个过大的提交
# - 需要修复某次提交引入的问题
```

**完整操作流程**：

```bash
# 启动交互式 rebase
git rebase -i HEAD~3

# 编辑器中修改为：
edit f7f3f6d feat: add user login functionality
pick 310154e fix: resolve authentication bug
pick a5f4a0d docs: update README

# 保存退出后，Git 会停在第一个提交
# 此时可以修改代码...

# 例如发现登录功能缺少验证逻辑
echo "// Add validation logic" >> AuthService.cs
git add AuthService.cs
git commit --amend          # 或 git rebase --continue

# 继续后续提交
git rebase --continue       # 自动应用剩余的 pick 提交
```

#### 操作四：squash（压缩）

```bash
# 将当前提交合并到前一个提交
s 310154e fix: resolve authentication bug

# 执行后会弹出编辑器，合并两个提交的信息
# 最终只生成一个提交

# 适用场景：
# - 将小的修正提交合并到主提交
# - 整理 WIP（Work In Progress）提交
# - PR 前清理提交历史
```

**squash 实战**：

```
原始提交历史：
1. feat: start working on auth
2. wip: add login form
3. wip: add validation
4. fix: typo in variable name
5. feat: complete auth module

目标：压缩为 1-2 个清晰的提交

rebase -i HEAD~5 编辑为：
pick 1a2b3c4 feat: start working on auth
s    2b3c4d5 wip: add login form
s    3c4d5e6 wip: add validation
s    4d5e6f7 fix: typo in variable name
pick 5e6f7g8 feat: complete auth module

结果：
1. feat(auth): implement user authentication module
   - Add login form with validation
   - Fix variable naming issues
2. feat: complete auth module
```

#### 操作五：fixup（静默压缩）

```bash
# 类似 squash，但丢弃被合并提交的消息
f 310154e fix: resolve authentication bug

# 与 squash 的区别：
# squash: 合并消息供你编辑
# fixup: 直接丢弃，使用前一个提交的消息

# 适用场景：
# - 明确属于前一个提交的小修补
# - 不想保留中间提交信息
```

#### 操作六：drop（删除）

```bash
# 完全删除该提交
d a5f4a0d docs: update README

# 适用场景：
# - 删除包含敏感信息的提交
# - 移除错误的提交
# - 删除实验性的无用提交
```

#### 操作七：reorder（重新排序）

```bash
# 通过调整 pick 的顺序来重新排列提交顺序
# 原始顺序：
pick A commit A
pick B commit B
pick C commit C

# 调整为：
pick C commit C      # C 移到最前面
pick A commit A
pick B commit B

# 注意：重新排序可能导致冲突！
# 如果 B 依赖 A 的变更，把 C 放前面可能出问题
```

### 2.3 常见 Interactive Rebase 场景

#### 场景一：PR 前整理提交

```bash
# 开发过程中可能产生很多小提交：
# 1. wip: start
# 2. wip: more work
# 3. fix: oops
# 4. wip: continue
# 5. fix: another fix
# 6. done!

# PR 前，整理成规范的提交：
git rebase -i origin/main

# 调整为：
pick 1 wip: start
s    2 wip: more work        # squash 到 1
fixup 3 fix: oops            # fixup 到前面的
s    4 wip: continue         # squash
fixup 5 fix: another fix     # fixup
pick 6 done!                 # 保留并 reword

# 结果可能是 2 个干净的提交
```

#### 场景二：拆分大提交

```bash
# 一个大提交包含了多个不相关的改动
# 需要拆分成多个独立提交

# 步骤：
git rebase -i HEAD~1

# 改为：
edit <hash> big commit message

# 然后 Git 停住，进行拆分：
git reset HEAD~

# 现在 staged area 为空，所有更改 unstaged
# 可以选择性添加和提交：

git add file1.cs
git commit -m "feat: add user service"

git add file2.cs
git commit -m "refactor: extract repository pattern"

git add file3.cs
git commit -m "test: add unit tests for user service"

# 继续完成 rebase
git rebase --continue
```

---

## 三、Cherry-pick / Stash / Bisect

### 3.1 Cherry-pick（摘樱桃）

将指定的提交"摘取"到当前分支。

```bash
# ===== 基本用法 =====

# 将单个提交应用到当前分支
git cherry-pick abc1234

# 将多个提交按顺序应用
git cherry-pick abc1234 def5678 ghi9012

# 不自动提交（先查看再决定）
git cherry-pick -n abc1234

# 应用提交但不包含原始作者信息
git cherry-pick --no-commit abc1234

# ===== 处理冲突 =====

# cherry-pick 时如果遇到冲突：
# 1. 手动解决冲突文件
# 2. git add 解决后的文件
# 3. git cherry-pick --continue

# 或者放弃：
# git cherry-pick --abort

# ===== 实际场景 =====

# 场景一：从 release 分支挑选 hotfix 到 main
git checkout main
git cherry-pick hotfix/abc1234

# 场景二：恢复误删除的提交
# 假设不小心 reset 掉了一个有用的提交
git reflog                      # 找到丢失提交的 hash
git cherry-pick <lost-hash>     # 恢复它

# 场景三：跨分支复制特定功能
git cherry-pick feature-x~3..feature-x
# 复制 feature-x 分支最近 3 个提交

# 场景四：选择性合并（不用 merge 整个分支）
git log feature-branch --oneline
# 选择需要的提交
git cherry-pick e5f6a7b
```

**注意事项**：

```
Cherry-pick 最佳实践：

✅ 推荐：
- 摘取明确的单个修复提交
- 从 release/hotfix 分支获取补丁
- 恢复意外丢失的工作

⚠️ 注意：
- 可能产生冲突（与 merge 类似）
- 摘取一系列相关提交时要小心顺序
- 摘取合并提交通常有问题

❌ 避免：
- 大规模 cherry-pick 替代 proper merge
- cherry-pick 已被 rebase 过的提交
```

### 3.2 Stash（暂存）高级用法

#### 基本 Stash 操作

```bash
# ===== 基础命令 =====

# 暂存当前所有更改（包括已暂存和未暂存的）
git stash

# 暂存并添加描述信息
git stash save "WIP: working on login feature"

# 查看 stash 列表
git stash list

# 恢复最近的 stash（并删除）
git stash pop

# 恢复特定的 stash
git stash apply stash@{2}

# 删除 stash
git stash drop stash@{0}

# 清空所有 stash
git stash clear

# 查看 stash 内容
git stash show -p stash@{0}
```

#### 多 Stash 管理

```bash
# ===== 创建带标签的 Stash =====

# 方式一：使用描述
git stash push -m "feature/login: WIP auth controller"
git stash push -m "bugfix/issue-42: partial fix for timeout"

# 方式二：按文件/目录暂存
# 只暂存特定文件
git stash push -m "WIP config" appsettings.json

# 只暂存特定目录下的更改
git stash push -m "WIP views" -- src/Pages/

# 包括未跟踪的文件
git stash push -u -m "include new files"

# 包括忽略的文件
git stash push -a -m "everything"

# ===== Stash 分支操作 =====

# 从 stash 创建新分支（推荐用于复杂的 stash 恢复）
git stash branch feature/stashed-work stash@{1}
# 这会创建新分支并在其上恢复 stash

# ===== 实用场景 =====

# 场景一：紧急切换任务
# 正在开发 feature/A，突然需要修 bug
git stash push -m "feature/A: almost done"
git checkout main
git checkout -b hotfix/urgent-bug
# ... 修复 bug ...
git checkout main && git merge hotfix/urgent-bug
git checkout feature/A
git stash pop

# 场景二：保存多种方案
# 尝试不同的实现方式
git stash push -m "approach-A: use Redis cache"
# ... 编写另一种实现 ...
git stash push -m "approach-B: use MemoryCache"

# 之后比较两种方案
git stash show -p stash@{0}   # 查看 approach-B
git stash show -p stash@{1}   # 查看 approach-A
git stash apply stash@{0}     # 采用 approach-B
git stash drop stash@{1}      # 丢弃 approach-A
```

**Stash 内部结构**：

```
stash@{0}: On main: feature/login: WIP auth controller
stash@{1}: On develop: bugfix/issue-42: partial fix
stash@{2}: On feature/api: experimental refactor

每个 Stash 包含：
- 工作区的更改（unstaged changes）
- 暂存区的更改（staged changes）
- 未跟踪的文件（如果使用了 -u）
- 当前分支信息和提交状态
- 创建时间和描述信息
```

### 3.3 Bisect（二分查找 Bug）

Git Bisect 使用二分查找算法快速定位引入 Bug 的提交。

```bash
# ===== 基本用法 =====

# 开始 bisect
git bisect start

# 标记当前版本是有问题的（bad）
git bisect bad

# 标记已知正常的版本（good）
git bisect good v2.0.0

# Git 会自动切换到中间版本
# 你测试该版本是否有问题：
# 如果有问题 → git bisect bad
# 如果没问题 → git bisect good

# Git 继续缩小范围...
# 直到定位到具体的有问题提交

# 结束 bisect
git bisect reset

# ===== 一键命令 =====

# 自动化 bisect（脚本返回 0=good, 非 0=bad）
git bisect start HEAD v1.0.0 --
git bisect run test-script.sh

# test-script.sh 示例:
#!/bin/bash
dotnet test --filter "FullyQualifiedName~BugTest"
# 返回退出码：0=通过(bad), 1=失败(good)
```

**完整实战示例**：

```bash
# 问题：用户报告登录功能在某个版本后失效了

# 1. 开始 bisect
$ git bisect start

# 2. 标记当前版本（确认有问题）
$ git bisect bad
Current revision has the bug

# 3. 标记之前正常工作的版本
$ git bisect good v2.1.0
Bisecting: 12 revisions left to test after this (roughly 3 steps)
[abc1234] Some intermediate commit

# 4. 测试当前检出的版本
$ dotnet test --filter LoginTests
# 测试通过 → 这个版本是好的
$ git bisect good
Bisecting: 6 revisions left... (about 2 steps)
[def5678] Another commit

# 5. 继续测试
$ dotnet test --filter LoginTests
# 测试失败 → 这个版本有问题
$ git bisect bad
Bisecting: 3 revisions left... (about 1 step)
[ghi9012] Another commit

# 6. 很快定位到了问题提交
$ dotnet test --filter LoginTests
# 失败
$ git bisect bad
Bisecting: 1 revision left...
[jkl3456] Commit that introduced the bug

# 7. 找到了！
$ git bisect good
jkl3456 is the first bad commit
commit jkl3456
Author: John Doe <john@example.com>
Date:   Mon Jan 15 10:30:00 2024 +0000

    refactor(auth): simplify token validation logic

# 查看这个提交改了什么
$ git show jkl3456

# 8. 重置
$ git bisect reset
```

**Bisect 进阶技巧**：

```bash
# 跳过无法测试的版本（如编译失败）
git bisect skip

# 可视化查看 bisect 日志
git bisect log
git bisect replay          # 重放之前的 bisect 会话

# 结合脚本自动化
git bisect start HEAD stable --
git bisect run sh -c '
  dotnet build --no-restore &&
  dotnet test --no-build --filter "Category=Regression"'
```

---

## 四、Reflog（引用日志）

### 4.1 Reflog 是什么

Reflog 记录了 HEAD 和分支引用的所有移动历史，是 Git 的"时光机"。即使提交看起来"丢失"了，Reflog 通常能帮你找回来。

```bash
# ===== 查看 Reflog =====

# 查看 HEAD 的移动历史
git reflog

# 查看特定分支的 reflog
git reflog show main
git reflog show feature/login

# 查看详细格式
git reflog --date=iso

# 输出示例：
# abc1234 (HEAD -> main) HEAD@{0}: commit: Add new feature
# def5678 HEAD@{1}: checkout: moving to main from feature
# ghi9012 HEAD@{2}: commit: Fix bug in auth
# jkl3456 HEAD@{3}: reset: moving to HEAD~2
# mno6789 HEAD@{4}: commit: Experimental change
# pqr8901 HEAD@{5}: checkout: moving to feature from main
```

### 4.2 用 Reflog 恢复丢失的工作

#### 场景一：恢复误 reset 的提交

```bash
# 假设你不小心执行了：
git reset --hard HEAD~3
# 丢失了 3 个提交！

# 不要慌，使用 reflog：
git reflog
# 找到 reset 之前的 HEAD 位置
# abc1234 HEAD@{0}: reset: moving to HEAD~3
# def5678 HEAD@{1}: commit: Third lost commit
# ghi9012 HEAD@{2}: commit: Second lost commit
# jkl3456 HEAD@{3}: commit: First lost commit
# mno6789 HEAD@{4}: commit: Some earlier commit

# 恢复到 reset 之前的状态
git reset --hard HEAD@{1}
# 或者直接指定提交
git reset --hard def5678
```

#### 场景二：恢复误删的分支

```bash
# 误删了分支
git branch -D important-feature

# 用 reflog 找到分支指向的提交
git reflog | grep important-feature
# 或者记得大概什么时候在哪个分支上
git reflog
# 找到类似这样的条目：
# abc1234 HEAD@{5}: checkout: moving from main to important-feature

# 重建分支
git branch important-feature abc1234
```

#### 场景三：恢复 cherry-pick 冲突后的状态

```bash
# cherry-pick 中途出了问题，想回到之前的状态
git reflog
# 找到 cherry-pick 之前的位置
git reset --hard HEAD@{n}
```

### 4.3 Reflog 维护

```bash
# Reflog 默认保留 90 天
# 可以调整保留时间：

# 设置过期时间（天）
git config --global gc.reflogExpire=90 days
git config --global gc.reflogExpireUnreachable=30 days

# 手动清理过期 reflog 条目
git gc --prune=now

# 查看即将被清理的条目
git reflog expire --expire=now --all
git reflog                  # 查看是否还有
```

---

## 五、Submodule（子模块）与 Worktree

### 5.1 Submodule 子模块

Submodule 允许你将一个 Git 仓库嵌入到另一个仓库中，常用于管理第三方库或共享组件。

```bash
# ===== 添加子模块 =====

# 添加子模块（默认放到仓库根目录下以子模块名命名的文件夹）
git submodule add https://github.com/org/shared-components.git

# 添加到指定目录
git submodule add https://github.com/org/shared-components.git src/libs/components

# 克隆包含子模块的项目
git clone --recursive https://github.com/my/project.git
# 或
git clone https://github.com/my/project.git
cd project
git submodule update --init --recursive

# ===== 更新子模块 =====

# 进入子模块目录更新
cd src/libs/components
git pull origin main
cd ../..

# 或从父项目更新所有子模块
git submodule update --remote

# 更新到指定提交
cd src/libs/components
git checkout specific-commit-hash
cd ../..
git add src/libs/components
git commit -m "chore: pin shared-components to v1.2.3"

# ===== 子模块日常操作 =====

# 查看子模块状态
git submodule status

# 同步子模块 URL（.gitmodules 变更后）
git submodule sync

# 删除子模块
# 步骤 1: 从 .gitmodules 中移除配置
# 步骤 2: git rm --cached <path>
# 步骤 3: rm -rf <path>
# 步骤 4: rm -rf .git/modules/<name>
# 步骤 5: commit
```

**.gitmodules 文件示例**：

```ini
[submodule "shared-components"]
    path = src/libs/components
    url = https://github.com/org/shared-components.git
    branch = main
```

### 5.2 Worktree（工作树）

Worktree 允许你同时检出多个分支到不同目录，无需频繁切换分支。

```bash
# ===== 创建 Worktree =====

# 基于 main 分支创建新的工作目录
git worktree add ../project-hotfix main

# 创建并同时创建新分支
git worktree add ../project-feature -b feature/new-dashboard

# 基于指定提交创建
git worktree add ../project-old-version abc1234

# ===== 管理 Worktree =====

# 列出所有 worktree
git worktree list

# 查看详情
git worktree list --porcelain

# 删除 worktree
git worktree remove ../project-hotfix

# 清理 stale（已删除路径）的 worktree
git worktree prune

# ===== 实用场景 =====

# 场景一：一边修 hotfix 一边继续开发 feature
git worktree add ../hotfix -b hotfix/critical-fix main
cd ../hotfix
# ... 修复 hotfix ...
# 同时原目录继续 feature 开发

# 场景二：基于旧版本构建发布包
git worktree add ../release-v1.0 tags/v1.0.0
cd ../release-v1.0
dotnet publish -c Release -o ./publish-v1.0

# 场景三：对比两个分支的差异
git worktree add ../compare-branch feature/x
# 在 IDE 中同时打开两个目录进行对比
```

---

## 六、Hook 脚本

### 6.1 Hook 类型与触发时机

```
Git Hooks 触发时机:

客户端 Hooks（本地触发）:
├── pre-commit          ← 提交前（检查代码、运行测试）
├── prepare-commit-msg  ← 编辑提交信息前（模板填充）
├── commit-msg          ← 提交信息完成后（验证格式）
├── post-commit         ← 提交后（通知）
├── pre-push            ← 推送前（运行完整测试套件）
├── pre-rebase          ← 变基前
├── post-checkout       ← 切换分支后
└── post-merge          ← 合并后

服务端 Hooks（远程触发）:
├── pre-receive         ← 接收推送前（权限检查）
├── update              ← 更新每个分支前
└── post-receive        ← 推送完成后（部署通知）
```

### 6.2 实用 Hook 示例

#### pre-commit：代码检查

```bash
#!/bin/sh
# .git/hooks/pre-commit

# .NET 项目 pre-commit hook

echo "Running pre-commit checks..."

# 检查代码格式
dotnet format --verify-no-quiet --verbosity diagnostic
if [ $? -ne 0 ]; then
    echo ""
    echo "❌ Code formatting issues found!"
    echo "Run 'dotnet format' to fix automatically."
    exit 1
fi

# 检查是否有大文件（>5MB）
MAX_SIZE=$(expr 5 \* 1024 \* 1024)
LARGE_FILES=$(find . -type f -size +${MAX_SIZE}c \
  ! -path './.git/*' \
  ! -path './bin/*' \
  ! -path './obj/*' \
  ! -path './node_modules/*')

if [ -n "$LARGE_FILES" ]; then
    echo ""
    echo "⚠️ Warning: Large files detected:"
    echo "$LARGE_FILES"
    echo "Consider using Git LFS for large files."
fi

# 检查敏感信息
if git diff --cached --name-only | xargs grep -l "password\|secret\|api_key" 2>/dev/null; then
    echo ""
    echo "❌ Potential sensitive information detected!"
    echo "Please remove secrets before committing."
    exit 1
fi

echo "✓ Pre-commit checks passed!"
exit 0
```

#### commit-msg：提交信息规范检查

```bash
#!/bin/sh
# .git/hooks/commit-msg

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# 检查提交信息不为空
if [ -z "$COMMIT_MSG" ]; then
    echo "❌ Empty commit message!"
    exit 1
fi

# 检查是否符合 Conventional Commits 规范
# 格式: type(scope): subject
if ! echo "$COMMIT_MSG" | grep -qE "^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?: .+"; then
    echo ""
    echo "❌ Invalid commit message format!"
    echo ""
    echo "Expected format:"
    echo "  type(scope): subject"
    echo ""
    echo "Types: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert"
    echo ""
    echo "Example:"
    echo "  feat(auth): add JWT token refresh"
    exit 1
fi

# 检查主题行长度（不超过 72 字符）
FIRST_LINE=$(echo "$COMMIT_MSG" | head -1)
if [ ${#FIRST_LINE} -gt 72 ]; then
    echo ""
    echo "⚠️ Subject line exceeds 72 characters (${#FIRST_LINE})"
    echo "Consider shortening it."
fi

# 检查是否有 Issue 引用（可选但推荐）
if ! echo "$COMMIT_MSG" | grep -qE "#[0-9]+"; then
    echo ""
    echo "ℹ️ Consider referencing an issue (e.g., Fixes #123)"
fi

exit 0
```

#### pre-push：推送前完整测试

```bash
#!/bin/sh
# .git/hooks/pre-push

echo "Running pre-push checks..."

# 获取将要推送的分支
REMOTE="$1"
URL="$2"

# 检查是否推送受保护分支
PROTECTED_BRANCHES="main master develop"
CURRENT_BRANCH=$(git symbolic-ref --short HEAD)

echo $PROTECTED_BRANCHES | tr ' ' '\n' | grep -q "^${CURRENT_BRANCH}$"
if [ $? -eq 0 ]; then
    echo ""
    echo "🛡️ Protected branch: $CURRENT_BRANCH"
    echo "Running full test suite..."

    # 运行完整测试
    dotnet test --configuration Release --no-build \
      --logger "console;verbosity=minimal"

    if [ $? -ne 0 ]; then
        echo ""
        echo "❌ Tests failed! Push aborted."
        exit 1
    fi

    echo "✓ All tests passed!"
else
    echo "Branch $CURRENT_BRANCH is not protected, skipping full tests."
fi

exit 0
```

#### 安装和管理 Hooks

```bash
# ===== 安装 Hook =====

# Hook 文件放在 .git/hooks/ 目录下
# 需要可执行权限（Linux/Mac）:
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg
chmod +x .git/hooks/pre-push

# ===== 团队共享 Hooks =====

# 方式一：使用 core.hooksPath 配置
mkdir -p .githooks
# 将 hooks 放入 .githooks/
git config core.hooksPath .githooks

# 方式二：使用工具管理
# Husky (Node.js 生态) 或 .NET 对应工具

# ===== 常用 Hook 管理工具 =====

# pre-commit framework (Python, 但语言无关)
pip install pre-commit

# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: dotnet-format
        name: dotnet format
        entry: dotnet format --verify-no-changes
        language: system
        files: \.cs$
      - id: detect-secrets
        name: Detect secrets
        entry: detect-secrets-hook
        language: system
```

### 6.3 Git 别名

```bash
# ===== 常用别名配置 =====

# 编辑全局配置
git config --global -e

# 在 [alias] 部分添加：

[alias]
    # 状态与日志
    st = status -sb
    co = checkout
    br = branch
    lg = log --oneline --graph --all --decorate
    ll = log --oneline --stat
    last = log -1 --stat
    recent = log --since='2 weeks ago' --oneline

    # 差异与查看
    d = diff
    ds = diff --staged
    dc = diff --cached
    bl = blame

    # 撤销操作
    unstage = reset HEAD --
    discard = checkout --
    amend = commit --amend --no-edit

    # 分支操作
    new = checkout -b
    delete = branch -D
    cleanup = "!git branch --merged | grep -v '\\*\\|main\\|master\\|develop' | xargs git branch -d"

    # 高级操作
    # 便捷的 interactive rebase
    rebase-last = "!f() { git rebase -i HEAD~${1:-3}; }; f"
    # 便捷的 squash
    squash = "!f() { git reset --soft HEAD~${1:-2}; }; f"

    # 查找功能
    whois = "!sh -c 'git log -i -1 --pretty=\"format:%an <%ae>\n\" --author=\"$1\"' -"
    whatis = show -s --pretty='tformat:%h (%s, %ad)' --date=short
    whereis = "!sh -c 'git log -i -1 --pretty=\"format:%H\" --grep=\"$1\"' -"

    # 实用工具
    # 显示漂亮的图形化日志
    graph = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim normal)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all

    # 快速查看今天的提交
    today = log --since='midnight' --author="$(git config user.name)" --oneline

    # 统计贡献
    contributions = shortlog -sn --no-merges

# ===== 使用示例 =====
git st                              # git status -sb
git co -b new-feature               # git checkout -b new-feature
git lg                               # 图形化日志
git unstage file.cs                  # 取消暂存
git discard file.cs                  # 丢弃修改
git rebase-last 5                    # 最近 5 个提交 interactive rebase
git squash 3                         # 压缩最近 3 个提交
git whois "john"                     # 查找 john 的提交
git today                            # 查看我今天的提交
```

---

## 七、常见棘手问题解决

### 7.1 合并冲突救援

```bash
# ===== 场景：复杂合并冲突 =====

# 方法一：使用合并工具
git mergetool                        # 配置好的可视化合并工具

# 方法二：接受某一方的全部更改
# 接受我们的版本（当前分支）
git checkout --ours .

# 接受他们的版本（要合并的分支）
git checkout --theirs .

# 方法三：逐个文件选择
git checkout --ours path/to/file.cs   # 特定文件用我们的
git checkout --theirs path/to/file.cs # 特定文件用他们的

# 方法四：放弃合并
git merge --abort                    # 回到合并前的状态

# 方法五：重新开始合并
git merge --abort
git merge feature-branch             # 重新尝试
```

**预防合并冲突的最佳实践**：

```
减少冲突的策略：

1. 频繁同步主分支
   git pull --rebase origin main 每天

2. 保持功能分支短命
   - 分支存活不超过 1-2 周
   - 及时合并或关闭

3. 关注修改区域
   - 团队成员避免同时修改同一文件
   - 大文件拆分为小模块

4. 使用良好的代码组织
   - 单一职责原则
   - 减少耦合

5. 合并前先更新
   git fetch origin
   git rebase origin/main
   再推送
```

### 7.2 历史重写（慎用！）

```bash
# ===== 场景：需要修改已推送的历史 =====

# ⚠️ 警告：以下操作会影响其他开发者！
# 必须通知所有协作者！

# 场景一：从历史中移除敏感信息
# 使用 git-filter-repo（推荐替代 BFG）
pip install git-filter-repo

# 移除包含密码的文件
git filter-repo --path secrets.json --invert-paths

# 或使用 BFG Repo Cleaner
java -jar bfg.jar --delete-files secrets.json repo.git

# 强制推送（必须通知团队！）
git push --force-with-lease origin main

# 场景二：修改提交者信息
git filter-repo --mailmap mailmap.txt

# mailmap.txt 格式:
# Correct Name <correct@email.com> <wrong@email.com>

# 场景三：拆分仓库（提取子目录为独立仓库）
git filter-repo --subdirectory-filter src/lib/my-component

# 场景四：合并多个仓库
git remote add other-repo https://github.com/other/repo.git
git fetch other-repo
git merge --allow-unrelated-histories other-repo/main
```

### 7.3 大文件清理

```bash
# ===== 问题：仓库体积过大 =====

# 检查大文件
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | sed -n 's/^blob //p' \
  | sort --numeric-sort --key=2 \
  | tail -20

# 或使用工具
du -sh .git/**/* | sort -rh | head -20

# ===== 解决方案 =====

# 方案一：Git LFS（推荐用于大文件管理）
# 安装 Git LFS
# Windows: 下载安装 https://git-lfs.github.com/
# Mac: brew install git-lfs
# Linux: sudo apt install git-lfs

# 跟踪大文件类型
git lfs install
git lfs track "*.psd" "*.mp4" "*.zip" "*.dll"
git add .gitattributes
git add .
git commit -m "Enable LFS for large files"

# 方案二：清理历史中的大文件
# 使用 BFG（更快）
java -jar bfg.jar --strip-blobs-bigger-than 5M some-repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 或使用 git-filter-repo
git filter-repo --strip-blobs-bigger-than 5M

# 清理后强制推送
git push --force-with-lease origin main

# 团队成员需要重新克隆仓库
```

### 7.4 其他常见问题速查

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| **detached HEAD** | 检出了某个提交而非分支 | `git checkout <branch-name>` |
| **空文件夹不被跟踪** | Git 不跟踪空目录 | 添加 `.gitkeep` 占位文件 |
| **文件名大小写问题** | Windows 不区分大小写 | `git config core.ignorecase false` |
| **换行符混乱** | Windows/Linux 换行符不同 | 配置 `core.autocrlf` |
| **提交太慢** | pre-commit hook 太耗时 | 优化 hook 或仅检查暂存文件 |
| **clone 太慢** | 仓库历史太大 | 使用 `--depth 1` 浅克隆 |
| **fetch/push 超时** | 网络问题或仓库太大 | 检查网络，增大 buffer (`git config http.postBuffer`) |
| **权限被拒绝** | SSH key 或凭证问题 | 检查 SSH 配置或更新 credential |

### 7.5 性能优化建议

```bash
# ===== 仓库性能优化 =====

# 垃圾回收（清理不可达对象）
git gc

# 激进垃圾回收（更彻底）
git gc --aggressive --prune=now

# 重新打包以提高性能
git repack -a -d -f --depth=250 --window=250

# 索引优化
git update-index -e

# 检查仓库完整性
git fsck --full

# ===== 大仓库优化 =====

# 启用文件系统监控（减少 git status 耗时）
git config core.fsmonitor true

# 部分克隆（只下载需要的内容）
git clone --filter=blob:none --sparse <repo>
git sparse-checkout set src/docs

# 配置并行操作
git config --global pack.threads 4
git config --global pack.indexVersion 2
```

---

## 八、高级命令速查卡

### 8.1 Rebase 系列

```bash
git rebase main                        # 变基到 main
git rebase -i HEAD~3                   # 交互式变基最近 3 个
git rebase --continue                  # 继续变基
git rebase --abort                     # 取消变基
git rebase --skip                      # 跳过当前提交
```

### 8.2 Cherry-pick / Stash / Bisect

```bash
git cherry-pick abc1234                # 摘取提交
git cherry-pick --no-commit abc1234    # 摘取但不提交
git stash                             # 暂存更改
git stash save "description"           # 带描述暂存
git stash list                         # 列出暂存
git stash pop                          # 恢复并删除
git stash apply stash@{2}              # 恢复指定暂存
git bisect start                       # 开始二分查找
git bisect good/bad                    # 标记好坏版本
git bisect reset                       # 结束查找
```

### 8.3 Reflog / Submodule / Worktree

```bash
git reflog                            # 查看 HEAD 历史
git reset --hard HEAD@{2}              # 回到 reflog 位置
git submodule add <url>                # 添加子模块
git submodule update --init --recursive # 初始化子模块
git worktree add ../dir -b branch      # 创建工作树
git worktree list                      # 列出工作树
git worktree remove ../dir             # 删除工作树
```

### 8.4 高级查询

```bash
git log -S "keyword" --source --all   # 搜索代码变更
git log --follow -p file.cs           # 追溯文件历史
git log --author="name" --since="1 month ago"  # 按条件搜索
git log --diff-filter=A               # 只看新增的文件
git shortlog -sn                      # 贡献统计
git whatchanged -p file.cs            # 文件详细变更历史
```

掌握这些 Git 高级技巧，你将能够从容应对各种复杂的版本控制场景。记住：高级功能虽然强大，但在团队协作中使用时务必谨慎，尤其是涉及历史重写的操作。
