# GitHub 操作指南

GitHub 是全球最大的代码托管平台和开发者社区，掌握 GitHub 的操作对于现代软件开发至关重要。本文全面介绍 GitHub 的核心功能使用方法，包括仓库管理、Pull Request 流程、Issues 管理、Actions 自动化以及 GitHub CLI 命令行工具。

## 一、GitHub 账号与仓库管理

### 1.1 账号注册与设置

#### 注册流程

```
GitHub 账号注册步骤：

1. 访问 https://github.com/signup
2. 输入邮箱地址
3. 创建密码（建议使用密码管理器）
4. 验证邮箱
5. 设置用户名（将成为你的唯一标识）
6. 选择是否接收邮件通知
7. 完成人机验证
8. 开始个性化设置
```

#### 个人资料优化

```markdown
## 推荐的 Profile 设置

- **头像**：清晰、专业的个人照片或品牌 Logo
- **姓名**：真实姓名或常用昵称
- **Bio**：简短介绍（160 字符内），如 "Full-stack .NET Developer | Open Source Enthusiast"
- **公司/组织**：当前工作单位
- **位置**：所在城市
- **个人网站/博客**：技术博客链接
- **Twitter/X**：社交媒体账号
```

#### README.md Profile 定制

```markdown
<!-- 在 your-username 仓库中创建 README.md -->

# Hi there, I'm [Your Name]! 👋

I'm a passionate developer who loves building things with .NET and web technologies.

## 🚀 About Me

- 🔭 I'm currently working on [project-name]
- 🌱 I'm currently learning [technology]
- 👯 I'm looking to collaborate on [topic]
- 💬 Ask me about [.NET Core, ASP.NET, C#]
- ⚡ Fun fact: [interesting fact]

## 🛠️ Tech Stack

![C#](assets/images/badges/csharp-badge.svg)
![.NET](assets/images/badges/dotnet-badge.svg)
![Azure](assets/images/badges/azure-badge.svg)
![Docker](assets/images/badges/docker-badge.svg)

## 📊 GitHub Stats

![GitHub Stats](assets/images/github/stats-card.svg)

## 📫 How to reach me

- Email: your.email@example.com
- LinkedIn: linkedin.com/in/your-profile
- Twitter: @your-handle
```

### 1.2 SSH 密钥配置

#### 生成并配置 SSH Key

```bash
# 检查是否已有 SSH 密钥
ls ~/.ssh/id_*.pub

# 生成新的 SSH 密钥（推荐 Ed25519）
ssh-keygen -t ed25519 -C "your.email@example.com"

# 或使用 RSA（兼容性更好）
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# 启动 SSH agent 并添加密钥
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 复制公钥到剪贴板（Windows PowerShell）
cat ~/.ssh/id_ed25519.pub | clip

# 或在 Linux/Mac 上
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
```

#### 在 GitHub 中添加 SSH Key

```
操作路径：
Settings → SSH and GPG keys → New SSH key

Title: My Laptop (2024)
Key type: Authentication Key
Key: [粘贴公钥内容]

添加后验证连接：
ssh -T git@github.com
# 应显示: Hi username! You've successfully authenticated...
```

#### SSH 配置文件（多密钥管理）

```bash
# ~/.ssh/config 文件内容

# GitHub 默认账户
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519

# GitHub 工作账户（可选）
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work

# 使用方式：
# git@github-work:org/repo.git
```

### 1.3 仓库创建与管理

#### 创建新仓库

```bash
# 方式一：通过 GitHub CLI
gh repo create my-awesome-project \
  --public \
  --description "An awesome .NET project" \
  --clone

# 方式二：通过 Git 命令（先创建本地再推送）
mkdir my-project && cd my-project
git init
echo "# My Project" > README.md
git add .
git commit -m "Initial commit"
git branch -M main
gh repo create my-project --public --source=. --remote=origin
git push -u origin main

# 方式三：网页创建后克隆
# 在 GitHub 网页上创建空仓库，然后：
git clone https://github.com/user/my-project.git
cd my-project
```

#### 仓库设置选项

| 设置项 | 说明 | 推荐 |
|--------|------|------|
| **Visibility** | Public / Private | 敏感项目选 Private |
| **Initialize with README** | 初始化 README | 是 |
| **Add .gitignore** | 忽略规则模板 | 选 .NET |
| **Choose license** | 开源协议 | MIT / Apache 2.0 |
| **Branch name** | 默认分支名 | main |

#### 仓库重要设置

```bash
# 通过 gh CLI 配置仓库
gh repo edit my-repo \
  --description "Updated description" \
  --homepage "https://my-app.example.com"

# 设置默认分支
gh repo edit my-repo --default-branch main

# 归档仓库
gh repo archive my-old-repo

# 删除仓库（不可恢复！）
gh repo delete my-repo --confirm
```

---

## 二、Pull Request 完整流程

### 2.1 PR 生命周期概览

```
┌──────────────────────────────────────────────────────────────┐
│                   Pull Request 生命周期                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Draft ──► Open ──► In Review ──► Approved ──► Merged        │
│   (草稿)    (开放)    (审查中)      (已批准)     (已合并)     │
│                                                              │
│  可能的状态转换:                                              │
│  ┌────────┐                                                   │
│  │ Closed │ ◄── 从任意状态可关闭                             │
│  └────────┘                                                   │
│                                                              │
│  关键节点:                                                    │
│  ├── CI Checks 运行                                         │
│  ├── Code Review 审查                                        │
│  ├── Changes Requested 要求修改                               │
│  └── Merge Commit 合并                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 创建 Pull Request

#### 使用 GitHub CLI

```bash
# 基本创建
gh pr create \
  --title "feat: add user authentication system" \
  --body "## Summary\nImplemented JWT-based authentication..."

# 完整参数示例
gh pr create \
  --title "Add user profile management" \
  --body "$(cat .github/pr-template.md)" \
  --base main \
  --head feature/user-profile \
  --reviewer john-doe,jane-smith \
  --assignee @me \
  --label "enhancement","documentation" \
  --milestone "v2.0.0" \
  --project "Q2 Roadmap"

# 创建 Draft PR（草稿状态，不触发完整审查）
gh pr create --draft \
  --title "WIP: new dashboard design" \
  --body "Work in progress, not ready for review"
```

#### PR 标题规范

```
格式: <type>: <description>

类型前缀:
├── feat:     新功能
├── fix:      Bug 修复
├── docs:     文档更新
├── style:    格式调整
├── refactor: 重构
├── perf:     性能优化
├── test:     测试相关
├── chore:    构建/工具变更
└── revert:   回滚提交

示例:
✅ feat(auth): implement OAuth2 login flow
✅ fix(api): resolve null reference in user controller
✅ docs(readme): update installation instructions
✅ refactor(db): extract repository pattern from DbContext
✅ perf(cache): add Redis caching layer for API responses
```

### 2.3 PR 审查流程

#### 作为作者：处理反馈

```bash
# 查看 PR 状态和评论
gh pr view 123 --comments
gh pr checks 123                    # 查看 CI 状态

# 更新 PR（继续在分支上开发）
git checkout feature/my-feature
# ... 修改代码 ...
git add .
git commit -m "address review feedback"
git push origin feature/my-feature
# PR 会自动更新

# 回复特定评论
gh pr comment 123 --body "Thanks for the review! Fixed the issue."

# 请求特定人员审查
gh pr edit 123 --add-reviewer senior-dev

# 将 PR 标记为准备审查（从 Draft 转为 Ready）
gh pr ready 123
```

#### 作为审查者：进行代码审查

```bash
# 查看待审查的 PR 列表
gh pr list --state open --review required

# 查看某个 PR 详情
gh pr view 123

# 在浏览器中打开 PR
gh pr view 123 --web

# 批准 PR
gh pr review 123 --approve --body "LGTM! Code looks good."

# 请求修改
gh pr review 123 --request-changes --body "Please address the following:\n1. ...\n2. ..."

# 一般性评论
gh pr review 123 --comment --body "Minor suggestion: consider using ..."

# 行内审查（在浏览器中操作更方便）
gh pr diff 123                     # 查看差异
gh pr checkout 123                 # 检出 PR 到本地测试
```

### 2.4 合并方式选择

```bash
# 三种合并方式的命令

# 1. Create a merge commit（保留完整历史）
gh pr merge 123 --merge

# 2. Squash and merge（压缩为一个提交，推荐）
gh pr merge 123 --squash \
  --delete-branch                  # 合并后删除远程分支

# 3. Rebase and merge（变基合并，线性历史）
gh pr merge 123 --rebase

# 合并后清理本地分支
git checkout main
git pull origin main
git branch -d feature/my-feature
```

**合并方式决策指南**：

| 场景 | 推荐方式 | 原因 |
|------|----------|------|
| 团队协作项目 | Squash Merge | 保持历史整洁 |
| 需要保留每个 PR 历史 | Merge Commit | 可追溯每个贡献者 |
| 追求完美线性历史 | Rebase Merge | 无合并节点 |
| 开源项目 | Squash Merge | 减少噪音提交 |

---

## 三、Issues 与项目管理

### 3.1 Issue 高效管理

#### 创建 Issue

```bash
# 基本创建
gh issue create \
  --title "Bug: Login fails with special characters in password" \
  --body "## Description\nWhen users include special characters..."

# 完整 Issue 创建
gh issue create \
  --title "Feature: Dark mode support" \
  --body "$(cat .github/issue-template.md)" \
  --label "enhancement","good first issue" \
  --assignee @me \
  --milestone "v2.1.0" \
  --project "Product Roadmap"

# 从终端快速创建（使用编辑器）
gh issue create --editor

# 从标准输入读取 body
echo "Found a bug in the auth module" | gh issue create \
  --title "Auth bug" \
  --body-file -
```

#### Issue 模板

```markdown
<!-- .github/ISSUE_TEMPLATE/bug_report.yml -->
name: Bug Report
description: File a bug report
labels: ["bug"]
assignees: []
body:
  - type: textarea
    id: description
    attributes:
      label: Description
      description: A clear description of the problem
      placeholder: Describe the issue here...
    validations:
      required: true
  - type: textarea
    id: steps
    attributes:
      label: Steps to Reproduce
      description: Steps to reproduce the behavior
      placeholder: |
        1. Go to '...'
        2. Click on '...'
        3. Scroll down to '...'
        4. See error
    validations:
      required: true
  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      description: What you expected to happen
  - type: textarea
    id: actual
    attributes:
      label: Actual Behavior
      description: What actually happened
  - type: input
    id: environment
    attributes:
      label: Environment
      description: OS, .NET version, browser, etc.
      placeholder: Windows 11, .NET 8.0, Chrome 120
  - type: textarea
    id: screenshots
    attributes:
      label: Screenshots
      description: Add screenshots if applicable
  - type: checkboxes
    id: checklist
    attributes:
      label: Confirmation
      options:
        - label: I have searched existing issues
          required: true
        - label: This is not a security vulnerability
          required: true
```

#### Issue 操作命令

```bash
# 搜索 Issues
gh issue list --search "login" --state open
gh issue list --author @me --state all
gh issue list --label "bug" --state open

# 查看 Issue 详情
gh issue view 42
gh issue view 42 --comments       # 包含评论
gh issue view 42 --web            # 在浏览器中打开

# 更新 Issue
gh issue edit 42 \
  --add-label "priority: high" \
  --add-assignee john-doe \
  --add-project "Sprint 5"

# 关闭 Issue
gh issue close 42

# 批量关闭
gh issue close 40 41 42 43

# 引用关联 Issue（在 PR 描述中）
# Fixes #42
# Closes #123
# Resolves #456
```

### 3.2 Projects 项目管理

#### Projects V2（新版）

```bash
# 创建项目
gh project create "Q2 Product Roadmap" \
  --owner org/team-name

# 添加 Issue 到项目
gh project item-add 123 \
  --project 1 \
  --url https://github.com/org/repo/issues/42

# 查看项目
gh project view 1 --web

# 项目字段管理
# Projects V2 支持自定义字段：Status、Priority、Assignee、Date 等
```

#### 看板视图使用技巧

```
Projects 看板列表示例:

┌──────────┬──────────┬──────────┬──────────┐
│  Backlog │  Todo    │ In Progress │ Done   │
├──────────┼──────────┼──────────┼──────────┤
│ #42      │ #45      │ #48      │ #38     │
│ #44      │ #46      │ #49      │ #39     │
│ #43      │ #47      │          │ #40     │
│          │          │          │ #41     │
└──────────┴──────────┴──────────┴──────────┘

最佳实践:
- 限制 WIP（在制品数量）：In Progress 不超过 3 个
- 每日站会时同步看板状态
- 大任务拆分为子 Issue
- Done 列的 Issue 定期归档
```

### 3.3 Milestones 里程碑

```bash
# 创建里程碑
gh milestone create "v2.0.0 Release" \
  --description "Major version release with new features" \
  --due-date "2024-06-30"

# 为 Issue 设置里程碑
gh issue edit 42 --milestone "v2.0.0"
gh issue edit 43 --milestone "v2.0.0"
gh issue edit 44 --milestone "v2.0.0"

# 查看里程碑进度
gh milestone view "v2.0.0"
# 显示: Open: 5, Closed: 10, Progress: 66%

# 编辑里程碑
gh milestone edit "v2.0.0" \
  --due-date "2024-07-15" \
  --title "v2.0.0 (Extended)"

# 列出所有里程碑
gh milestone list --state open
```

---

## 四、GitHub Actions 入门

### 4.1 基础概念

```
GitHub Actions 核心组件：

Workflow（工作流）:
├── 定义在 .github/workflows/*.yml 文件中
├── 由事件触发（push, PR, schedule, manual）
├── 包含一个或多个 Job（作业）

Job（作业）:
├── 在 Runner（运行器）上执行
├── 同一 Job 的 Step 顺序执行
├── 不同 Job 默认并行执行

Step（步骤）:
├── 基本单位，执行具体动作
├── 可以是 shell 命令或 Action

Action（动作）:
├── 可复用的任务单元
├── 来自社区或自己编写

Runner（运行器）:
├── GitHub-hosted（免费额度内）
├── Self-hosted（自托管服务器）
```

### 4.2 第一个 Workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: '8.0.x'

jobs:
  build-and-test:
    name: Build and Test
    runs-on: ubuntu-latest

    steps:
      # 步骤 1: 检出代码
      - name: Checkout code
        uses: actions/checkout@v4

      # 步骤 2: 安装 .NET SDK
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      # 步骤 3: 缓存依赖
      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: ~/.nuget/packages
          key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
          restore-keys: |
            ${{ runner.os }}-nuget-

      # 步骤 4: 还原依赖
      - name: Restore dependencies
        run: dotnet restore

      # 步骤 5: 构建
      - name: Build
        run: dotnet build --no-restore --configuration Release

      # 步骤 6: 运行测试
      - name: Run tests
        run: dotnet test --no-build --configuration Release \
          --logger "trx;LogFileName=test-results.trx" \
          --collect:"XPlat Code Coverage"

      # 步骤 7: 上传测试结果
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: '**/*.trx'

      # 步骤 8: 代码质量检查
      - name: Format check
        run: dotnet format --verify-no-changes
```

### 4.3 常用 Workflow 模板

#### 自动部署到 Azure

```yaml
# .github/workflows/deploy.yml
name: Deploy to Azure

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Publish
        run: dotnet publish ./src/WebApi/WebApi.csproj \
          -c Release \
          -o ./publish

      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ./publish
```

#### Docker 镜像构建与推送

```yaml
# .github/workflows/docker.yml
name: Build Docker Image

on:
  push:
    tags: ['v*']

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}

      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### 4.4 Secrets 管理

```bash
# 通过 CLI 设置 Secrets
gh secret set AZURE_CONNECTION_STRING \
  --repo owner/repo \
  --body "DefaultEndpointsProtocol=https;..."

# 设置环境 Secret
gh secret set PROD_DB_PASSWORD \
  --env production \
  --body "super-secret-password"

# 列出 Secrets（只能看到名称，看不到值）
gh secret list --repo owner/repo

# 删除 Secret
gh secret delete OLD_SECRET --repo owner/repo
```

**Secret 类型**：

| 类型 | 作用域 | 用途 |
|------|--------|------|
| Repository Secrets | 单个仓库 | API Keys, 连接字符串 |
| Environment Secrets | 环境（如 production） | 生产数据库密码 |
| Organization Secrets | 组织 | 共享的服务凭证 |

---

## 五、Gists 代码片段分享

### 5.1 Gist 基础操作

```bash
# 创建公开 Gist
gh gist create <<'EOF'
// Hello World in C#
Console.WriteLine("Hello, World!");
EOF

# 创建私有 Gist
gh gist create --private <<'EOF'
// Internal utility function
private string GenerateToken() { ... }
EOF

# 从文件创建 Gist
gh gist create snippet.cs config.json

# 带描述的 Gist
gh gist create --desc "C# Extension Methods" extensions.cs

# 列出我的 Gists
gh gist list

# 查看特定 Gist
gh gist view abc123def456

# 克隆 Gist（作为 Git 仓库）
gh gist clone abc123def456

# 删除 Gist
gh gist delete abc123def456

# 编辑 Gist
gh gist edit abc123def456 --file updated-snippet.cs
```

### 5.2 Gist 最佳用途

```
Gist 适用场景:

✅ 分享代码片段（Stack Overflow 风格）
✅ 存储配置文件模板
✅ 收集常用的脚本片段
✅ 技术演示的 PoC 代码
✅ .dotfiles 片段分享
✅ 博客文章配套代码

❌ 不适合:
- 完整的项目代码 → 用仓库
- 敏感信息 → 绝对不要放 Gist
- 需要版本管理的代码 → 用仓库
- 大型文件 → Gist 有大小限制
```

---

## 六、GitHub CLI (gh) 命令速查

### 6.1 安装与认证

```bash
# ===== 安装 =====

# Windows (winget)
winget install GitHub.cli

# Windows (Chocolatey)
choco install gh

# macOS (Homebrew)
brew install gh

# 验证安装
gh --version

# ===== 认证 =====

# 登录（交互式）
gh auth login

# 使用浏览器认证（推荐）
gh auth login --web

# 使用 Token 认证
gh auth login --with-token < token.txt

# 检查登录状态
gh auth status

# 注销
gh auth logout
```

### 6.2 核心命令一览

```bash
# ===== 仓库操作 =====
gh repo create <name>              # 创建仓库
gh repo clone <repo>               # 克隆仓库
gh repo view                        # 查看仓库信息
gh repo fork <repo>                # Fork 仓库
gh repo edit                        # 编辑仓库设置
gh repo list                        # 列出仓库

# ===== Issue 操作 =====
gh issue create                     # 创建 Issue
gh issue list                       # 列出 Issue
gh issue view <number>              # 查看 Issue
gh issue close <number>             # 关闭 Issue
gh issue comment <number>           # 评论 Issue
gh issue edit <number>              # 编辑 Issue
gh issue lock <number>              # 锁定 Issue

# ===== PR 操作 =====
gh pr create                        # 创建 PR
gh pr list                          # 列出 PR
gh pr view <number>                 # 查看 PR
gh pr checkout <number>             # 检出 PR
gh pr merge <number>                # 合并 PR
gh pr review <number>               # 审查 PR
gh pr diff <number>                 # 查看 PR 差异
gh pr comment <number>              # 评论 PR
gh pr close <number>                # 关闭 PR
gh pr ready <number>                # 标记为准备好审查
gh pr status                        # 查看当前分支的 PR 状态

# ===== Actions 操作 =====
gh workflow list                    # 列出 Workflows
gh workflow run <workflow>          # 触发 Workflow
gh workflow view <workflow>         # 查看 Workflow
gh run list                         # 列出运行记录
gh run view <run-id>                # 查看运行详情
gh run watch <run-id>               # 监视运行状态
gh run rerun <run-id>               # 重新运行

# ===== Release 操作 =====
gh release create <tag>             # 创建 Release
gh release list                     # 列出 Releases
gh release view <tag>               # 查看 Release
gh release download <tag>           # 下载 Release 资产

# ===== Gist 操作 =====
gh gist create                      # 创建 Gist
gh gist list                        # 列出 Gists
gh gist view <id>                   # 查看 Gist
gh gist clone <id>                  # 克隆 Gist
gh gist delete <id>                 # 删除 Gist

# ===== 其他实用命令 =====
gh api <endpoint>                   # 调用 GitHub REST API
gh search repos "<query>"           # 搜索仓库
gh search issues "<query>"          # 搜索 Issue
gh browse                            # 在浏览器中打开
gh config list                      # 查看配置
```

### 6.3 高级用法示例

#### 批量操作

```bash
# 批量关闭旧的 stale issues
gh issue list --state open --limit 100 \
  --json number,title,createdAt \
  --jq '.[] | select(.createdAt < "2024-01-01") | .number' \
  | xargs -I {} gh issue close {}

# 批量为 PR 添加标签
gh pr list --state open --json number \
  --jq '.[].number' \
  | xargs -I {} gh pr edit {} --add-label "needs-review"

# 导出仓库统计信息
gh api repos/$GITHUB_REPOSITORY \
  --jq '{stars: .stargazers_count, forks: .forks_count, issues: .open_issues_count}'
```

#### API 直接调用

```bash
# 获取仓库信息
gh api repos/owner/repo

# 创建一个 Issue（直接调用 API）
gh api repos/owner/repo/issues \
  --method POST \
  -f title="API created issue" \
  -f body="Created via gh cli"

# 搜索代码
gh api search/code \
  -f q="DbContext repo:owner/repo" \
  --jq '.items[] | {path: .path, html_url: .html_url}'

# 获取用户信息
gh api users/octocat --jq '{login, name, bio, public_repos}'
```

#### 自定义别名

```bash
# 添加常用别名
gh alias set prs 'pr list --state open'
gh alias set mine 'issue list --author @me'
gh alias set co 'pr checkout'
gh alias set rv 'pr view --web'

# 查看别名列表
gh alias list

# 使用别名
gh prs                              # 等同于 gh pr list --state open
gh mine                             # 等同于 gh issue list --author @me
gh co 123                           # 等同于 gh pr checkout 123
```

---

## 七、团队与组织管理

### 7.1 组织基础设置

```bash
# 创建组织
gh org create my-org \
  --display-name "My Awesome Team"

# 查看组织信息
gh org view my-org

# 列出组织成员
gh org view my-org --json members \
  --jq '.members[].login'

# 列出组织的仓库
gh repo list my-org --limit 50
```

### 7.2 权限模型

```
GitHub 权限层级:

Organization Level:
├── Owner（所有者）
│   └── 完全控制，账单访问
├── Admin（管理员）
│   └── 管理成员和仓库
├── Member（成员）
│   └── 访问公共仓库和被授权的私有仓库
└── Guest（外部协作者）
    └── 仅限被明确邀请的仓库

Repository Level:
├── Admin（管理员）
│   └── 完全控制该仓库
├── Maintain（维护者）
│   └── 代码管理 + Issue/PR 管理
├── Write（写入）
│   └── 推送代码 + 管理 Issue/PR
├── Triage（分类）
│   └── 管理 Issue/PR 标签
└── Read（只读）
    └── 拉取和克隆
```

### 7.3 团队管理

```bash
# 创建团队
gh team create backend-devs \
  --org my-org \
  --privacy closed

# 添加成员到团队
gh team add-member backend-devs --member john-doe

# 向团队添加仓库权限
gh team add-repo backend-devs \
  --org my-org \
  --repo my-api \
  --permission push

# 列出团队成员
gh team list-members backend-devs --org my-org
```

---

## 八、安全最佳实践

### 8.1 仓库安全检查清单

```bash
# 启用依赖审查
gh api repos/owner/repo/dependency-graph-config \
  --method PUT \
  -f use_detect_all=true

# 启用代码扫描（CodeQL）
# Settings → Code security → Setup code scanning

# 启用 secret scanning
# Settings → Code security → Secret scanning

# 查看安全告警
gh api repos/owner/repo/code-scanning/alerts \
  --jq '.[] | {rule_id: .rule.id, state: .state}'

# 查看 Dependabot Alerts
gh api repos/owner/repo/dependabot/alerts \
  --jq '.[] | {package: .security_advisory.package.name, severity: .security_advisory.severity}'
```

### 8.2 安全配置建议

| 配置项 | 建议 | 原因 |
|--------|------|------|
| Branch Protection | 启用 | 防止直接推送到主分支 |
| Required Reviews | 至少 1 人 | 保证代码质量 |
| CI Required | 必须通过 | 自动化质量保障 |
| Signed Commits | 可选 | 防止伪造提交 |
| Secret Scanning | 启用 | 自动检测泄露的密钥 |
| Dependabot | 启用 | 自动更新依赖 |
| 2FA Requirement | 强烈推荐 | 账户安全 |
| Private by Default | 推荐 | 最小权限原则 |

掌握 GitHub 的这些核心功能，可以显著提升团队的开发效率和协作体验。从基础的仓库管理到高级的 Actions 自动化，GitHub 提供了完整的 DevOps 工具链支持。
