# Git 基础命令速查

Git 是目前最流行的分布式版本控制系统，掌握 Git 命令是每个开发者的必备技能。本文系统介绍 Git 的核心概念、常用命令、配置方法以及 .NET 项目中的最佳实践。

## 一、Git 核心概念图解

### 1.1 四大区域

```
┌─────────────────────────────────────────────────────────────────┐
│                        Git 工作流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    git add     ┌──────────┐    git commit        │
│  │          │ ─────────────► │          │ ─────────────►       │
│  │  工作区   │                │  暂存区   │                     │
│  │ (Working │ ◄───────────── │ (Staging │ ◄─────────────       │
│  │  Directory)│  git restore  │  Area)   │  git reset           │
│  │          │                │  (Index) │                      │
│  └──────────┘                └──────────┘                      │
│                                      │                          │
│                               git commit                       │
│                                      ▼                          │
│                              ┌──────────┐                      │
│                              │ 本地仓库  │                      │
│                              │(Local    │                      │
│                              │ Repo)    │                      │
│                              └──────────┘                      │
│                                      │                          │
│                               git push / git pull              │
│                                      ▼                          │
│                              ┌──────────┐                      │
│                              │ 远程仓库  │                      │
│                              │(Remote   │                      │
│                              │ Repo)    │                      │
│                              └──────────┘                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 文件状态转换

```
                    git add               git commit
Untracked ────────► Staged ──────────────► Committed
   │                  │                       │
   │                  │     git restore        │
   │                  │ ◄──────────────────────┤
   │                  │  (从暂存区移除)         │
   │                  │                       │
   │  编辑文件         │                       │
   ├──────────────────┤                       │
   │                  ▼                       │
   │             Modified ◄──────────────────┤
   │                  │  (修改已提交的文件)     │
   │                  │                       │
   └──────────────────┘

状态说明：
- Untracked: 新文件，Git 尚未跟踪
- Staged: 已添加到暂存区，等待提交
- Modified: 已修改，但未暂存
- Committed: 已提交到本地仓库
```

---

## 二、初始化与克隆

### 2.1 初始化仓库

```bash
# 在当前目录创建新的 Git 仓库
git init

# 创建并指定仓库名称
git init my-project

# 初始化裸仓库（用于服务器端）
git init --bare server-repo.git
```

**初始化后的目录结构**：

```
my-project/
├── .git/              # Git 内部数据目录
│   ├── HEAD           # 当前分支引用
│   ├── config         # 仓库配置
│   ├── description    # 仓库描述
│   ├── hooks/         # 钩子脚本
│   ├── info/          # 仓库信息
│   ├── objects/       # 对象存储（压缩的文件内容）
│   └── refs/          # 引用存储（分支、标签等）
└── ...                # 项目文件
```

### 2.2 克隆仓库

```bash
# 基本克隆
git clone https://github.com/user/repo.git

# 克隆到指定目录
git clone https://github.com/user/repo.git my-folder

# 浅克隆（只获取最新一次提交，节省空间和时间）
git clone --depth 1 https://github.com/user/repo.git

# 克隆特定分支
git clone -b develop https://github.com/user/repo.git

# 克隆时使用 SSH 协议
git clone git@github.com:user/repo.git

# 克隆私有仓库（需要认证）
git clone https://<token>@github.com/user/private-repo.git
```

**克隆协议对比**：

| 协议 | 格式 | 优点 | 缺点 |
|------|------|------|------|
| HTTPS | `https://...` | 防火墙友好，简单 | 每次需输入密码 |
| SSH | `git@...` | 安全，可配置密钥 | 需配置 SSH Key |
| Git | `git://...` | 快速 | 无认证，不安全 |

---

## 三、添加与提交

### 3.1 基础操作

```bash
# ===== 添加文件到暂存区 =====

# 添加单个文件
git add filename.cs

# 添加所有变更文件
git add .

# 添加所有已跟踪文件的变更（不包括新文件）
git add -u

# 交互式选择要添加的内容
git add -p

# 添加特定类型的文件
git add "*.cs"
git add "src/**/*.cs"

# ===== 提交更改 =====

# 带消息的提交
git commit -m "feat: add user authentication"

# 打开编辑器编写详细提交信息
git commit

# 暂存所有已跟踪文件并提交
git commit -am "fix: resolve login issue"

# 修改最近一次提交（合并到上一次）
git commit --amend -m "updated message"

# 空提交（用于触发 CI 或文档）
git commit --allow-empty -m "chore: trigger CI pipeline"
```

### 3.2 提交信息规范（Conventional Commits）

```
格式: <type>(<scope>): <subject>

类型 (type):
├── feat:     新功能
├── fix:      Bug 修复
├── docs:     文档更新
├── style:    代码格式（不影响功能）
├── refactor: 重构（非新功能、非修复）
├── perf:     性能优化
├── test:     测试相关
├── chore:    构建/工具/辅助工具变动
├── ci:       CI 配置变更
└── revert:   回滚提交

示例：
feat(auth): add JWT token refresh mechanism
fix(api): handle null reference in user service
docs(readme): update installation steps
refactor(database): extract repository pattern
test(user): add unit tests for validation logic
perf(caching): implement Redis cache layer
ci(github): add automated testing workflow
```

### 3.3 查看状态与差异

```bash
# ===== 查看工作区状态 =====
git status                        # 详细状态
git status -s                     # 简短状态
git status -u                     # 显示未跟踪文件
git status --ignored              # 包括被忽略的文件

# 输出示例说明：
# M  file.cs      # 已暂存的修改
# M file.cs       # 未暂存的修改
# ?? newfile.cs   # 未跟踪的新文件
# D  deleted.cs   # 已删除（已暂存）
# R  old -> new   # 重命名（已暂存）

# ===== 查看差异 =====

# 工作区 vs 暂存区（未暂存的修改）
git diff

# 暂存区 vs 最新提交（已暂存的修改）
git diff --staged
git diff --cached                 # 同上

# 工作区 vs 指定提交
git diff HEAD
git diff HEAD~1

# 只查看某个文件的差异
git diff src/Program.cs

# 比较两个分支
git diff main..feature-branch

# 比较两个提交
git diff abc1234 def5678

# 统计差异（显示行数变化）
git diff --stat
git diff --shortstat

# ===== 查看提交历史 =====

# 基本日志
git log

# 单行简洁模式
git log --oneline

# 图形化展示分支合并
git log --oneline --graph --all

# 显示最近 N 条记录
git log -5
git log -10 --oneline

# 显示每次提交的详细差异
git log -p

# 搜索提交信息
git log --grep="fix"

# 搜索代码变更
git log -S "functionName" --source --all

# 查看某文件的历史
git log --follow -p src/File.cs

# 自定义输出格式
git log --pretty=format:"%h %ad | %s (%an)" --date=short
```

**自定义日志别名推荐**：

```bash
# 推荐添加到 ~/.gitconfig
[alias]
    lg = log --oneline --graph --all --decorate
    ll = log --oneline --stat
    l = log --pretty=format:"%C(yellow)%h%Creset %C(green)%ad%Creset %s" --date=format:"%Y-%m-%d %H:%M"
```

---

## 四、分支操作

### 4.1 分支基础

```bash
# ===== 分支管理 =====

# 查看本地分支
git branch

# 查看所有分支（包括远程）
git branch -a
git branch -av                   # 显示最后提交信息

# 创建新分支
git branch feature/login

# 创建并切换到新分支
git checkout -b feature/login
# 或使用新语法（Git 2.23+）
git switch -c feature/login

# 切换分支
git checkout main
git switch main                   # 新语法（推荐）

# 删除分支
git branch -d feature/login      # 安全删除（已合并）
git branch -D feature/login      # 强制删除（未合并）

# 重命名分支
git branch -m old-name new-name
git branch -M old-name new-name  # 强制重命名
```

### 4.2 合并与变基

```bash
# ===== 合并 (Merge) =====

# 将 feature 分支合并到当前分支
git merge feature/login

# 合并但不创建合并提交（快进）
git merge --ff-only feature/login

# 总是创建合并提交（即使可以快进）
git merge --no-ff feature/login

# 取消合并（合并冲突或后悔了）
git merge --abort

# ===== 变基 (Rebase) =====

# 将当前分支变基到 main 上
git rebase main

# 交互式变基（修改最近3个提交）
git rebase -i HEAD~3

# 取消变基
git rebase --abort

# 继续变基（解决冲突后）
git rebase --continue

# 跳过当前提交
git rebase --skip
```

**Merge vs Rebase 选择指南**：

```
场景判断：
├── 个人本地分支 → 用 rebase 保持历史整洁
├── 公共分支 → 必须用 merge（不要 rebase 已推送的提交）
├── 团队协作 → 遵循团队约定
└── 不确定 → 使用 merge（更安全）
```

### 4.3 远程分支操作

```bash
# ===== 远程分支 =====

# 查看远程分支
git branch -r

# 获取远程分支信息
git fetch origin

# 获取远程所有分支的最新信息
git fetch --all

# 基于远程分支创建本地分支
git checkout -b feature origin/feature
git switch -c feature origin/feature

# 推送新分支到远程
git push -u origin feature/new-feature

# 删除远程分支
git push origin --delete feature/old-feature
git push origin :feature/old-feature  # 旧写法

# 跟踪远程分支
git branch --set-upstream-to=origin/main main
git branch -u origin/main            # 简写
```

---

## 五、远程操作

### 5.1 远程仓库管理

```bash
# ===== 远程仓库配置 =====

# 查看远程仓库
git remote -v

# 添加远程仓库
git remote add origin https://github.com/user/repo.git

# 修改远程仓库地址
git remote set-url origin https://github.com/user/new-repo.git

# 移除远程仓库
git remote remove origin

# 重命名远程仓库
git remote rename origin upstream

# 查看远程仓库详细信息
git remote show origin
```

### 5.2 推送与拉取

```bash
# ===== 推送 (Push) =====

# 推送到默认远程分支
git push

# 推送到指定远程和分支
git push origin main

# 首次推送并设置上游分支
git push -u origin main

# 强制推送（慎用！会覆盖远程历史）
git push -f
git push --force-with-lease          # 更安全的强制推送

# 推送所有分支
git push --all origin

# 推送标签
git push origin --tags
git push origin v1.0.0               # 推送单个标签

# ===== 拉取 (Pull) =====

# 拉取并合并（fetch + merge）
git pull

# 拉取并变基（fetch + rebase）
git pull --rebase

# 从指定远程拉取
git pull origin main

# 拉取但不合并（仅获取数据）
git fetch
git fetch origin

# 获取所有远程分支
git fetch --all

# 清理已删除的远程分支引用
git fetch --prune
git prune                           # 同上
```

### 5.3 协作工作流

```bash
# 典型的 Pull Request 工作流

# 1. 从主分支创建特性分支
git checkout main
git pull origin main
git checkout -b feature/add-dashboard

# 2. 开发并提交
# ... 编写代码 ...
git add .
git commit -m "feat(dashboard): add admin dashboard page"

# 3. 推送到远程
git push -u origin feature/add-dashboard

# 4. 在 GitHub/GitLab 上创建 Pull Request

# 5. 处理审查反馈（继续在分支上修改）
git add .
git commit -m "address review feedback"
git push

# 6. PR 合并后，同步主分支
git checkout main
git pull origin main

# 7. 删除已合并的特性分支
git branch -d feature/add-dashboard
git push origin --delete feature/add-dashboard
```

---

## 六、撤销操作

### 6.1 撤销工作区修改

```bash
# ===== restore (Git 2.23+ 推荐) =====

# 撤销工作区的修改（丢弃更改）
git restore filename.cs

# 撤销所有工作区修改
git restore .

# 从暂存区恢复到工作区（取消暂存）
git restore --staged filename.cs
git restore -S filename.cs           # 等价写法

# 从指定提交恢复文件
git restore source~1 filename.cs

# ===== 传统命令（仍可用） =====

# 等同于 git restore（撤销工作区修改）
git checkout -- filename.cs

# 等同于 git restore --staged（取消暂存）
git reset HEAD filename.cs
```

### 6.2 重置提交

```bash
# ===== reset 命令详解 =====

# soft：保留更改在工作区和暂存区（回退提交但保留修改）
git reset --soft HEAD~1

# mixed（默认）：保留更改在工作区（取消暂存）
git reset HEAD~1
git reset --mixed HEAD~1

# hard：完全丢弃所有更改（危险！）
git reset --hard HEAD~1

# 重置到指定提交
git reset --hard abc1234

# 重置到远程状态（丢弃本地所有提交）
git reset --hard origin/main

# 实用场景
git reset --soft HEAD~1             # 修改最后一次提交信息
git reset HEAD~1                    # 取消最后一次提交但保留修改
```

**三种 Reset 模式对比**：

```
假设有三次提交: A -> B -> C (HEAD)

--soft reset 到 A:
  暂存区: B 和 C 的更改都在 ✓
  工作区: 干净
  提交历史: 只有 A

--mixed reset 到 A (默认):
  暂存区: 空 ✗
  工作区: B 和 C 的更改都在 ✓
  提交历史: 只有 A

--hard reset 到 A:
  暂存区: 空 ✗
  工作区: 干净 ✗
  提交历史: 只有 A
  ⚠️ B 和 C 的更改永久丢失！
```

### 6.3 回滚提交（Revert）

```bash
# ===== revert（安全地撤销） =====

# 回滚最近一次提交（创建一个新的撤销提交）
git revert HEAD

# 回滚指定提交
git revert abc1234

# 回滚多个提交（按时间倒序）
git revert HEAD~3..HEAD

# 不自动提交（先查看再决定）
git revert -n HEAD

# 回滚合并提交
git revert -m 1 <merge-commit-hash>
```

**Reset vs Revert 对比**：

| 特性 | Reset | Revert |
|------|-------|--------|
| 安全性 | 可能丢失代码 | 安全，创建新提交 |
| 历史影响 | 重写历史 | 追加新提交 |
| 适用范围 | 本地未推送的提交 | 已推送的公共提交 |
| 团队协作 | 仅限个人分支 | 可安全用于任何情况 |

---

## 七、标签管理

### 7.1 标签操作

```bash
# ===== 创建标签 =====

# 创建轻量标签（仅引用某个提交）
git tag v1.0.0

# 创建附注标签（包含元信息，推荐）
git tag -a v1.0.0 -m "Release version 1.0.0"

# 给指定提交打标签
git tag -a v0.9.0 abc1234 -m "Beta release"

# ===== 查看标签 =====

# 列出所有标签
git tag

# 查看标签详情
git show v1.0.0

# 搜索标签
git -l "v1.*"

# ===== 操作标签 =====

# 切换到标签指向的状态
git checkout v1.0.0

# 基于标签创建分支
git checkout -b release/v1.0.0 v1.0.0

# 删除标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0
git push origin :refs/tags/v1.0.0

# 推送标签
git push origin v1.0.0              # 单个标签
git push origin --tags              # 所有标签
```

### 7.2 语义化版本规范

```
格式: MAJOR.MINOR.PATCH

MAJOR: 不兼容的 API 变更
MINOR: 向下兼容的功能新增
PATCH: 向下兼容的问题修复

预发布版本:
- v1.0.0-alpha.1    # 内部测试版
- v1.0.0-beta.1     # 公开测试版
- v1.0.0-rc.1       # 候选发布版
- v1.0.0            # 正式发布版

.NET 项目建议：
- 使用 MinVer 或 Nerdbank.GitVersioning 自动生成版本号
- 标签与 NuGet 包版本保持一致
```

---

## 八、.NET 项目专用 .gitignore

### 8.1 完整模板

```gitignore
## ================================
## .NET 专用 Git Ignore 规则
## ================================

# === 用户特定文件 ===
*.rsuser
*.suo
*.user
*.userosscache
*.sln.docstates

# === 构建结果 ===
[Dd]ebug/
[Dd]ebugPublic/
[Rr]elease/
[Rr]eleases/
x64/
x86/
[Ww][Ii][Nn]32/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# === Visual Studio ===
.vs/
*.vsconfig

# === MSTest 测试结果 ===
[Tt]est[Rr]esult*/
[Bb]uild[Ll]og.*

# === NUnit ===
*.VisualState.xml
TestResult.xml
nunit-*.xml

# === .NET Core ===
project.lock.json
project.fragment.lock.json
artifacts/

# === ASP.NET Core ===
# 如果使用 IIS Express
.vs/config/applicationhost.config

# === 发布文件夹 ===
publish/

# === NuGet 包 ===
*.nupkg
*.snupkg
packages/
project.lock.json
project.assets.json

# === Rider ===
.idea/
*.sln.iml
*.iml
out/

# === VS Code ===
.vscode/*
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
*historyfiles

# === macOS ===
.DS_Store
.AppleDouble
.LSOverride

# === Windows ===
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.cab
*.msi
*.msix
*.msm
*.msp
*.lnk

# === 临时文件 ===
*.tmp
*.temp
*.swp
*~
*.bak
*.orig

# === 敏感信息（手动添加）===
# appsettings.Production.json
# *.pfx
# *.key
# secrets.json
# .env
# .env.local

# === 数据库文件 ===
*.db
*.sqlite
*.mdb
*.accdb

# === 日志文件 ===
*.log
logs/
```

### 8.2 全局 .gitignore 设置

```bash
# 设置全局忽略规则（适用于所有项目）
git config --global core.excludesfile ~/.gitignore_global

# 将以下内容写入 ~/.gitignore_global
# .DS_Store
# Thumbs.db
# *.swp
# *~
```

---

## 九、Git 配置

### 9.1 用户配置

```bash
# ===== 身份信息设置 =====

# 设置用户名（必填）
git config --global user.name "Your Name"

# 设置邮箱（必填）
git config --global user.email "your.email@example.com"

# 为特定项目设置不同身份
cd /path/to/project
git config user.name "Work Name"
git config user.email "work@company.com"

# ===== 查看配置 =====

# 查看所有配置
git config --list

# 查看特定配置项
git config user.name
git config --global core.editor

# 查看配置来源
git config --show-origin --list
```

### 9.2 核心配置选项

```bash
# ===== 编辑器配置 =====
git config --global core.editor "code --wait"          # VS Code
git config --global core.editor "notepad"               # 记事本
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar"

# ===== 默认分支名 =====
git config --global init.defaultBranch main

# ===== 换行符处理（Windows 推荐）=====
git config --global core.autocrlf true                  # 提交时转 LF→CRLF，检出时 CRLF→LF
# Linux/Mac:
# git config --global core.autocrlf input               # 提交时转 CRLF→LF，检出时不转换

# ===== 颜色输出 =====
git config --global color.ui auto                       # 自动着色
git config --global color.status auto
git config --global color.branch auto
git config --global color.diff auto
git config --global color.interactive auto

# ===== 别名配置 =====
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.last 'log -1'
git config --global alias.unstage 'reset HEAD --'
git config --global alias.visual '!git log --oneline --graph --all'

# ===== 安全相关 =====
# 拒绝合并非快进
git config --global merge.ff only

# 重新基准默认行为
git config --global pull.rebase true

# 凭证缓存（避免频繁输入密码）
git config --global credential.helper manager-core      # Windows
git config --global credential.helper cache --timeout=3600  # 缓存1小时
```

### 9.3 配置文件位置

```
配置优先级（从低到高）：
1. /etc/gitconfig              # 系统（所有用户）
2. ~/.gitconfig                # 全局（当前用户）
3. .git/config                 # 仓库级（当前项目）

# 编辑对应级别的配置
git config --system ...        # 系统
git config --global ...        # 全局
git config --local ...         # 仓库
```

---

## 十、图形化工具对比

### 10.1 主流 GUI 工具

| 工具 | 平台 | 特点 | 价格 |
|------|------|------|------|
| **GitKraken** | Win/Mac/Linux | 界面美观，跨平台，集成 GitHub/GitLab | 免费（Pro 收费） |
| **SourceTree** | Win/Mac | Atlassian 出品，免费易用 | 免费 |
| **Fork** | Win/Mac | 轻量快速，界面简洁 | 免费 |
| **TortoiseGit** | Windows | 资源管理器集成，右键操作 | 免费 |
| **GitHub Desktop** | Win/Mac | GitHub 官方，简化操作 | 免费 |
| **VS Code Git** | 跨平台 | 编辑器内置，无需切换 | 免费 |
| **Git Extensions** | Windows | 功能全面，高度可定制 | 免费 |
| **SmartGit** | 跨平台 | 商业软件，功能强大 | 付费（非商用免费） |

### 10.2 推荐组合

```
新手入门：
├── VS Code 内置 Git（日常操作）
└── GitHub Desktop（可视化理解）

进阶用户：
├── VS Code + GitLens 扩展（高效编码）
└── Git Kraken（复杂操作可视化）

专业开发者：
├── 命令行为主（90% 操作）
├── VS Code Git（快速查看差异）
└── Git Kraken（解决复杂冲突）
```

---

## 十一、实用技巧集合

### 11.1 高频操作速查表

| 任务 | 命令 |
|------|------|
| 开始新项目 | `git init && git add . && git commit -m "init"` |
| 修复上次提交 | `git commit --amend` |
| 查看我改了什么 | `git diff` |
| 查看将要提交什么 | `git diff --staged` |
| 撤销文件修改 | `git restore <file>` |
| 取消暂存 | `git restore --staged <file>` |
| 删除未跟踪的文件 | `git clean -fd` |
| 查看谁改了某行 | `git blame <file>` |
| 搜索代码内容 | `git log -S "keyword"` |
| 暂存部分文件 | `git add -p` |
| 查看分支图 | `git log --oneline --graph --all` |

### 11.2 常见问题速查

| 问题 | 解决方案 |
|------|----------|
| 提交后发现漏了文件 | `git add forgotten.cs && git commit --amend --no-edit` |
| 推送被拒绝（非快进） | `git pull --rebase && git push` |
| 合并冲突 | 手动编辑冲突文件 → `git add . && git commit` |
| 错误提交到错误的分支 | `git reset --soft HEAD~1` → 切换正确分支 → `git commit` |
| 想放弃所有本地更改 | `git reset --hard HEAD` |
| 恢复误删的文件 | `git restore <file>` 或 `git checkout HEAD -- <file>` |
| 大文件导致仓库臃肿 | 使用 BFG 或 `git filter-repo` 清理 |
| 忘记提交信息写了什么 | `git show --no-patch --format=%s HEAD` |
| 想看某个文件的历史版本 | `git show HEAD~5:path/to/file` |

### 11.3 每日工作流清单

```bash
# === 开始一天的工作 ===

# 1. 切换到主分支并拉取最新代码
git checkout main
git pull origin main

# 2. 创建/切换到工作分支
git checkout -b feature/my-task
# 或继续昨天的工作
git checkout feature/my-task

# 3. 检查状态
git status

# === 开发过程中 ===

# 4. 定期提交（小步快跑）
git add .
git commit -m "feat: implement login form validation"

# 5. 推送备份
git push origin feature/my-task

# === 结束工作时 ===

# 6. 确保所有更改已提交
git status

# 7. 推送到远程
git push

# 8. 如需创建 PR，去 GitHub/GitLab 操作
```

---

## 十二、命令完整速查卡

### 基础命令

```bash
# 仓库
git init                          # 初始化
git clone <url>                   # 克隆
git status                        # 状态

# 暂存与提交
git add <file>                    # 暂存
git add .                         # 暂存全部
git commit -m "msg"               # 提交
git commit -am "msg"              # 暂存并提交

# 分支
git branch                        # 列出分支
git branch <name>                 # 创建分支
git checkout <branch>             # 切换分支
git checkout -b <name>            # 创建并切换
git switch -c <name>              # 创建并切换（新）
git branch -d <name>              # 删除分支
git merge <branch>                # 合并分支

# 远程
git remote -v                     # 查看远程
git fetch                         # 获取
git pull                          # 拉取
git push                          # 推送

# 撤销
git restore <file>                # 撤销修改
git restore --staged <file>       # 取消暂存
git reset HEAD~1                  # 撤销提交（保留修改）
git reset --hard HEAD~1           # 撤销提交（丢弃修改）
git revert <commit>               # 安全回滚

# 查看
git diff                          # 差异
git diff --staged                 # 暂存区差异
git log                           # 历史
git log --oneline --graph         # 图形化历史
git show <commit>                 # 查看提交详情
git blame <file>                  # 追溯责任

# 标签
git tag -a v1.0 -m "msg"          # 创建标签
git tag -l                        # 列出标签
git push origin --tags            # 推送标签

# 清理
git clean -fd                     # 清理未跟踪文件
git gc                            # 垃圾回收优化
git prune                         # 清理不可达对象
```

掌握这些 Git 命令和概念，足以应对绝大多数日常开发场景。建议初学者从基础命令开始练习，逐步熟悉后再学习高级技巧。记住：多动手实践是掌握 Git 的最佳途径。
