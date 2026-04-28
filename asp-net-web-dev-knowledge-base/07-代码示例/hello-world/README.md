# Hello World 个人网站项目

> **对应教程**：[[入门篇/03-第一个应用]]
> **难度等级**：⭐ 入门级 | **预计耗时**：2-3小时
> **适用人群**：零基础或刚接触 ASP.NET Core 的开发者

---

## 项目概述

这是你的**第一个 ASP.NET Core MVC 个人展示网站**，一个功能完整但结构清晰的入门项目。通过构建这个项目，你将掌握 MVC 架构的核心概念、Razor 视图引擎的使用方法，以及现代 Web 开发的基本工作流程。

本项目模拟了一个**个人技术博客/作品集网站**的雏形，包含首页展示、关于页面、技能卡片等典型模块。它不是简单的"Hello World"页面，而是一个具备真实布局系统、组件化开发和静态资源管理能力的微型 Web 应用。

**核心价值**：
- 理解 MVC（Model-View-Controller）设计模式的实际运作方式
- 掌握 Razor 语法和视图渲染机制
- 学会使用 Bootstrap 5 快速构建响应式界面
- 建立正确的 ASP.NET Core 项目组织习惯

---

## 技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **ASP.NET Core** | 8.0 | Web 应用框架（LTS 长期支持版本） |
| **Razor Views** | - | 服务端视图引擎，混合 HTML 和 C# 代码 |
| **Bootstrap 5** | 5.3.x | 前端 UI 框架，提供响应式栅格系统和组件 |
| **C#** | 12 | 编程语言（最新稳定版） |
| **.NET SDK** | 8.0+ | 开发工具链 |

**为什么选择这个技术组合？**
- ASP.NET Core 8.0 是微软当前推荐的 LTS 版本，生产环境稳定可靠
- Razor Views 是学习服务端渲染的最佳起点，比 Blazor 更易上手
- Bootstrap 5 可以让你专注于后端逻辑，无需深入 CSS 细节
- C# 12 提供了现代化的语法特性，代码更简洁优雅

---

## 功能清单

### 核心页面

#### 1️⃣ 首页（Home/Index）
- **个人介绍区域**：头像、姓名、职位描述、简短简介
- **技能卡片网格**：使用 Bootstrap Card 组件展示 6-8 个技术技能
  - 每张卡片包含：技能图标、技能名称、熟练度进度条、简短描述
- **特色功能区**：3-4 个 Feature Box 展示项目亮点
  - 使用 Partial View 实现组件化
  - 支持图标 + 标题 + 描述的三段式布局

#### 2️⃣ 关于页面（Home/About）
- 详细个人背景介绍
- 教育经历时间线
- 工作经验列表
- 联系方式信息（使用 ContactInfo Partial View）

#### 3️⃣ 布局系统（Layout）
- **_Layout.cshtml**：主布局文件
  - HTML head 区域（meta 标签、CSS 引用）
  - 导航栏（Navbar）：包含 Logo、导航链接（首页/关于）、响应式折叠菜单
  - 主内容区域：`@RenderBody()` 占位符
  - 页脚（Footer）：版权信息、社交链接
  - JavaScript 引用区域
- **@RenderSection**：定义可选/必需的 Section
  - `@RenderSection("Scripts", required: false)`：允许页面注入自定义脚本
  - `@RenderSection("Styles", required: false)`：允许页面注入自定义样式

#### 4️⃣ Partial View 组件库
- **SkillCard.cshtml**：可复用的技能卡片组件
  - 参数：图标类名、技能名称、熟练度百分比（0-100）、描述文本
  - 内部实现：Bootstrap Card + Progress Bar
- **ContactInfo.cshtml**：联系方式组件
  - 参数：邮箱、电话、GitHub 地址、LinkedIn 地址
  - 内部实现：图标列表 + 超链接
- **FeatureBox.cshtml**：特色功能盒子
  - 参数：图标、标题、描述
  - 内部实现：带阴影的卡片容器

#### 5️⃣ 静态资源管理
- **CSS 目录**（wwwroot/css/）：
  - site.css：全局自定义样式（字体、颜色变量、通用工具类）
- **JS 目录**（wwwroot/js/）：
  - site.js：全局交互逻辑（平滑滚动、导航高亮）
- **Images 目录**（wwwroot/images/）：
  - avatar.jpg：个人头像
  - hero-bg.jpg：首页 Hero 区域背景图
  - skills/：技能相关图标（可使用 Font Awesome 或 SVG）

---

## 目录结构

```
hello-world/
├── HelloWorld.sln                          # 解决方案文件
├── HelloWorld.Web/                         # 主 Web 项目
│   ├── HelloWorld.Web.csproj               # 项目配置文件
│   ├── Program.cs                          # 应用程序入口点
│   ├── appsettings.json                    # 配置文件
│   ├── Properties/
│   │   └── launchSettings.json             # 启动配置（IIS Express/Kestrel）
│   │
│   ├── Controllers/
│   │   └── HomeController.cs              # 首页控制器（Index/About/Error）
│   │
│   ├── Views/
│   │   ├── Home/
│   │   │   ├── Index.cshtml              # 首页视图
│   │   │   └── About.cshtml              # 关于页视图
│   │   ├── Shared/
│   │   │   ├── _Layout.cshtml           # 主布局文件
│   │   │   ├── _ViewStart.cshtml        # 视图启动配置
│   │   │   ├── _ViewImports.cshtml      # 全局 using 导入
│   │   │   ├── Error.cshtml            # 错误页视图
│   │   │   └── Components/             # Partial View 组件目录
│   │   │       ├── SkillCard.cshtml    # 技能卡片组件
│   │   │       ├── ContactInfo.cshtml  # 联系方式组件
│   │   │       └── FeatureBox.cshtml   # 特色功能盒子
│   │   └── _ViewImports.cshtml         # 全局命名空间导入
│   │
│   ├── Models/
│   │   ├── SkillViewModel.cs           # 技能卡片数据模型
│   │   ├── FeatureViewModel.cs         # 特色功能数据模型
│   │   └── ContactViewModel.cs         # 联系信息数据模型
│   │
│   ├── wwwroot/
│   │   ├── css/
│   │   │   ├── bootstrap.min.css       # Bootstrap 样式（CDN 或本地）
│   │   │   └── site.css               # 自定义样式
│   │   ├── js/
│   │   │   ├── bootstrap.bundle.min.js # Bootstrap JS（含 Popper）
│   │   │   └── site.js                # 自定义脚本
│   │   ├── images/
│   │   │   ├── avatar.jpg             # 个人头像
│   │   │   ├── hero-bg.jpg            # 首页背景图
│   │   │   └── skills/               # 技能图标文件夹
│   │   │       ├── csharp.svg
│   │   │       ├── aspnetcore.svg
│   │   │       ├── sqlserver.svg
│   │   │       ├── git.svg
│   │   │       ├── docker.svg
│   │   │       └── azure.svg
│   │   └── lib/                       # 静态库文件（如使用 LibMan）
│   │       └── bootstrap/
│   │           └── dist/
│   │
│   └── wwwroot/favicon.ico            # 网站图标
│
├── .gitignore                           # Git 忽略配置
├── README.md                            # 本文件
└── docs/                                # 项目文档（可选）
    └── architecture.md                  # 架构说明文档
```

---

## 运行步骤

### 前置条件检查清单

在开始之前，请确保你的开发环境满足以下要求：

- [ ] 已安装 [.NET 8.0 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)（运行 `dotnet --version` 检查版本 ≥ 8.0.0）
- [ ] 已安装 [Visual Studio 2022](https://visualstudio.microsoft.com/)（社区版即可）或 [VS Code](https://code.visualstudio.com/) + C# Dev Kit 扩展
- [ ] 推荐安装 SQL Server（本项目不强制要求，后续项目会用到）

### 步骤 1：创建项目骨架（2分钟）

打开终端（PowerShell 或 CMD），执行以下命令：

```bash
# 创建解决方案文件夹
mkdir hello-world
cd hello-world

# 创建解决方案文件
dotnet new sln -n HelloWorld

# 创建 ASP.NET Core MVC 项目
dotnet new mvc -n HelloWorld.Web -o HelloWorld.Web --no-https

# 将项目添加到解决方案
dotnet sln add HelloWorld.Web/HelloWorld.Web.csproj
```

**命令解释**：
- `--no-HTTPS`：开发环境跳过 HTTPS 配置，简化本地调试
- `-o`：指定输出目录

### 步骤 2：添加前端依赖（3分钟）

在 `HelloWorld.Web` 目录下执行：

```bash
cd HelloWorld.Web

# 方案 A：使用 LibMan 安装 Bootstrap（推荐）
dotnet tool install -g Microsoft.Web.LibraryManager.Cli
libman init
libman install bootstrap@5.3.2 -d wwwroot/lib/bootstrap --provider cdnjs

# 方案 B：直接下载文件到 wwwroot（离线方案）
# 手动从 https://getbootstrap.com/ 下载并解压到 wwwroot/lib/bootstrap
```

或者，你也可以直接在 `_Layout.cshtml` 中使用 CDN 引入（适合快速原型开发）：

```html
<!-- 在 <head> 中添加 -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- 在 </body> 前添加 -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
```

### 步骤 3：编写核心代码（30-60分钟）

按照以下顺序逐步实现各个组件：

#### 3.1 创建数据模型（Models/）

**SkillViewModel.cs**：
```csharp
namespace HelloWorld.Web.Models;

public class SkillViewModel
{
    public string IconClass { get; set; } = string.Empty;  // Font Awesome 图标类名
    public string Name { get; set; } = string.Empty;        // 技能名称
    public int Proficiency { get; set; }                     // 熟练度 (0-100)
    public string Description { get; set; } = string.Empty; // 描述文本
}
```

**FeatureViewModel.cs**：
```csharp
namespace HelloWorld.Web.Models;

public class FeatureViewModel
{
    public string Icon { get; set; } = string.Empty;
    public string Title { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
}
```

#### 3.2 创建控制器（Controllers/HomeController.cs）

```csharp
using Microsoft.AspNetCore.Mvc;
using HelloWorld.Web.Models;

namespace HelloWorld.Web.Controllers;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        // 准备技能列表数据
        var skills = new List<SkillViewModel>
        {
            new() { IconClass = "fab fa-microsoft", Name = "C#", Proficiency = 90, Description = ".NET 开发主力语言" },
            new() { IconClass = "fas fa-server", Name = "ASP.NET Core", Proficiency = 85, Description = "Web 框架核心技能" },
            new() { IconClass = "fas fa-database", Name = "SQL Server", Proficiency = 75, Description = "关系型数据库管理" },
            new() { IconClass = "fab fa-git-alt", Name = "Git", Proficiency = 80, Description = "版本控制工具" },
            new() { IconClass = "fab fa-docker", Name = "Docker", Proficiency = 70, Description = "容器化部署" },
            new() { IconClass = "fab fa-azure", Name = "Azure", Proficiency = 65, Description = "云服务平台" }
        };

        // 准备特色功能数据
        var features = new List<FeatureViewModel>
        {
            new() { Icon = "fas fa-rocket", Title = "快速开发", Description = "ASP.NET Core 提供高效的开发体验" },
            new() { Icon = "fas fa-shield-alt", Title = "安全可靠", Description = "内置安全特性和最佳实践" },
            new() { Icon = "fas fa-expand-arrows-alt", Title = "跨平台", Description = "支持 Windows/Linux/macOS" },
            new() { Icon = "fas fa-cloud", Title = "云原生", Description = "完美适配云环境和容器部署" }
        };

        ViewBag.Skills = skills;
        ViewBag.Features = features;

        return View();
    }

    public IActionResult About()
    {
        return View();
    }

    public IActionResult Error()
    {
        return View();
    }
}
```

#### 3.3 创建 Partial View 组件（Views/Shared/Components/）

**SkillCard.cshtml**：
```html
@model HelloWorld.Web.Models.SkillViewModel

<div class="card h-100 shadow-sm">
    <div class="card-body text-center">
        <i class="@Model.IconClass fa-3x text-primary mb-3"></i>
        <h5 class="card-title">@Model.Name</h5>
        <div class="progress mb-2">
            <div class="progress-bar bg-success" role="progressbar"
                 style="width: @Model.Proficiency%"
                 aria-valuenow="@Model.Proficiency"
                 aria-valuemin="0"
                 aria-valuemax="100">
            </div>
        </div>
        <small class="text-muted">@Model.Description</small>
    </div>
</div>
```

**FeatureBox.cshtml**：
```html
@model HelloWorld.Web.Models.FeatureViewModel

<div class="col-md-6 col-lg-3 mb-4">
    <div class="card border-0 h-100 text-center feature-box">
        <div class="card-body">
            <div class="feature-icon mb-3">
                <i class="@Model.Icon fa-2x text-primary"></i>
            </div>
            <h5 class="card-title">@Model.Title</h5>
            <p class="card-text text-muted">@Model.Description</p>
        </div>
    </div>
</div>
```

#### 3.4 创建主布局（Views/Shared/_Layout.cshtml）

关键部分示例：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - 我的世界</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <!-- 导航栏 -->
    <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
        <div class="container">
            <a class="navbar-brand" asp-controller="Home" asp-action="Index">我的世界</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse d-sm-inline-flex justify-content-between">
                <ul class="navbar-nav flex-grow-1">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-controller="Home" asp-action="Index">首页</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-controller="Home" asp-action="About">关于</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>

    <!-- 主内容区域 -->
    <div class="container">
        @RenderBody()
    </div>

    <!-- 页脚 -->
    <footer class="border-top footer text-muted mt-auto py-3">
        <div class="container text-center">
            <p>&copy; 2026 - 我的世界 - <a asp-controller="Home" asp-action="Privacy">隐私政策</a></p>
        </div>
    </footer>

    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

#### 3.5 创建首页视图（Views/Home/Index.cshtml）

```html
@{
    ViewData["Title"] = "首页";
}

<!-- Hero 区域 -->
<div class="hero-section text-center py-5 mb-5 bg-light rounded">
    <img src="~/images/avatar.jpg" alt="我的头像" class="rounded-circle mb-3" width="150" height="150" />
    <h1 class="display-4">你好，我是开发者！</h1>
    <p class="lead text-muted">热爱技术，专注 .NET 全栈开发</p>
    <a asp-controller="Home" asp-action="About" class="btn btn-primary btn-lg">了解更多 &raquo;</a>
</div>

<!-- 技能卡片区域 -->
<h2 class="text-center mb-4">技术技能</h2>
<div class="row row-cols-1 row-cols-md-3 g-4 mb-5">
    @foreach (var skill in ViewBag.Skills)
    {
        <partial name="Components/SkillCard" model="skill" />
    }
</div>

<!-- 特色功能区 -->
<h2 class="text-center mb-4">为什么选择我？</h2>
<div class="row">
    @foreach (var feature in ViewBag.Features)
    {
        <partial name="Components/FeatureBox" model="feature" />
    }
</div>
```

### 步骤 4：运行与调试（2分钟）

```bash
# 进入项目目录
cd HelloWorld.Web

# 运行项目（默认监听 http://localhost:5000）
dotnet run

# 或者指定端口
dotnet run --urls "http://localhost:5050"
```

**预期结果**：
- 浏览器自动打开 `http://localhost:5000`
- 看到带有个人介绍、技能卡片、特色功能的首页
- 点击导航栏的"关于"可以跳转到关于页面
- 移动端访问时导航栏自动折叠为汉堡菜单

### 步骤 5：优化与发布（10分钟）

#### 5.1 添加自定义样式（wwwroot/css/site.css）

```css
/* 全局样式 */
:root {
    --primary-color: #0d6efd;
    --secondary-color: #6c757d;
    --success-color: #198754;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Hero 区域 */
.hero-section {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
}

.hero-section h1,
.hero-section p {
    color: white;
}

/* 特色盒子悬停效果 */
.feature-box:hover {
    transform: translateY(-5px);
    transition: transform 0.3s ease;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15)!important;
}

/* 技能卡片动画 */
.card:hover {
    transform: scale(1.02);
    transition: transform 0.2s ease;
}

/* 平滑滚动 */
html {
    scroll-behavior: smooth;
}
```

#### 5.2 发布生产版本

```bash
# 发布为独立部署（包含运行时）
dotnet publish -c Release -o ../publish --self-contained true -r win-x64

# 发布为依赖框架部署（需要目标机器安装 .NET Runtime）
dotnet publish -c Release -o ../publish
```

发布后的文件位于 `publish/` 目录，可直接部署到 IIS 或 Docker。

---

## 学习要点

通过完成本项目，你将掌握以下 **8 个关键知识点**：

### 1. MVC 架构模式 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐

MVC 是 ASP.NET Core 的核心架构模式，将应用分为三个职责明确的部分：

- **Model（模型）**：数据和业务逻辑（如 `SkillViewModel`）
- **View（视图）**：用户界面呈现（如 `.cshtml` 文件）
- **Controller（控制器）**：处理用户请求并协调 Model 和 View（如 `HomeController`）

**工作流程**：
```
用户请求 → 路由匹配 → Controller Action → 处理逻辑 → 选择 View → 返回 HTML 响应
```

**关联教程**：[[入门篇/01-MVC基础概念]]

**实际应用场景**：任何需要服务端渲染的 Web 应用都基于此模式

---

### 2. Razor 视图引擎 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

Razor 允许你在 HTML 中嵌入 C# 代码，使用 `@` 符号作为转换标记：

**常用语法**：
```html
<!-- 输出变量值 -->
<p>@Model.Name</p>

<!-- 代码块 -->
@{
    var greeting = "Hello";
    var time = DateTime.Now.Hour;
}

<!-- 条件判断 -->
@if (time < 12)
{
    <span>上午好！</span>
}
else
{
    <span>下午好！</span>
}

<!-- 循环 -->
@foreach (var item in Model.Items)
{
    <li>@item.Name</li>
}

<!-- 辅助方法 -->
<input asp-for="Email" class="form-control" />
<a asp-controller="Home" asp-action="About">关于</a>
```

**关联教程**：[[入门篇/02-Razor语法入门]]

---

### 3. Layout 布局系统 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐

Layout 解决了多个页面共享相同结构的问题（导航栏、页脚、CSS/JS 引用）：

**核心机制**：
- **_Layout.cshtml**：定义页面骨架
- **@RenderBody()**：子视图内容的插入位置
- **@RenderSection()**：定义可选/必需的内容区块（如自定义脚本）

**优势**：
- 避免重复代码（DRY 原则）
- 统一修改一处即全局生效
- 支持嵌套布局（高级用法）

**关联教程**：[[入门篇/04-Layout与PartialView]]

---

### 4. Partial View 组件化 ⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

Partial View 是可复用的视图片段，类似于前端的"组件"概念：

**本项目中的实践**：
- `SkillCard.cshtml`：封装技能卡片的 HTML 结构和样式
- `FeatureBox.cshtml`：封装特色功能盒子的展示逻辑
- `ContactInfo.cshtml`：封装联系方式的显示格式

**调用方式**：
```html
<!-- 传递模型对象 -->
<partial name="Components/SkillCard" model="@skill" />

<!-- 不传递模型（纯静态内容） -->
<partial name="Components/Copyright" />
```

**好处**：
- 提高代码复用性
- 降低维护成本（修改一处，全局生效）
- 分离关注点（每个组件职责单一）

**关联教程**：[[入门篇/04-Layout与PartialView#PartialView部分]]

---

### 5. 静态资源管理 ⭐⭐
**重要性**：★★★☆☆ | **难度**：⭐

ASP.NET Core 通过 `wwwroot` 目录统一管理静态文件（CSS、JS、图片、字体等）：

**目录约定**：
```
wwwroot/
├── css/          # 样式表
├── js/           # JavaScript 文件
├── images/       # 图片资源
├── lib/          # 第三方库（Bootstrap、jQuery 等）
└── favicon.ico   # 网站图标
```

**引用方式**：
```html
<!-- 使用 ~ 符号表示 wwwroot 根目录 -->
<link rel="stylesheet" href="~/css/site.css" />
<img src="~/images/avatar.jpg" alt="头像" />
<script src="~/js/site.js"></script>
```

**性能建议**：
- 生产环境启用静态文件压缩（Brotli/Gzip）
- 使用 CDN 加速第三方库加载
- 图片使用 WebP 格式减小体积

**关联教程**：[[入门篇/05-静态文件处理]]

---

### 6. Bootstrap 5 响应式设计 ⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

Bootstrap 提供了一套完整的响应式栅格系统和 UI 组件：

**栅格系统核心**：
- 容器：`.container`（固定宽度）/ `.container-fluid`（全宽）
- 行：`.row`
- 列：`.col-*-*`（设备类型 - 占比）
  - `col-sm-*`：≥576px（手机横屏）
  - `col-md-*`：≥768px（平板）
  - `col-lg-*`：≥992px（桌面）
  - `col-xl-*`：≥1200px（大屏）

**本项目使用案例**：
```html
<!-- 技能卡片：移动端1列，平板3列，桌面3列 -->
<div class="row row-cols-1 row-cols-md-3 g-4">
    <div class="col">卡片1</div>
    <div class="col">卡片2</div>
    <div class="col">卡片3</div>
</div>
```

**关联教程**：[[入门篇/06-Bootstrap快速上手]]

---

### 7. 依赖注入与服务配置 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

虽然本项目没有显式使用 DI（依赖注入），但 ASP.NET Core 的整个框架都建立在 DI 之上：

**Program.cs 中的隐式 DI**：
```csharp
var builder = WebApplication.CreateBuilder(args);

// 添加 MVC 服务（内部注册了大量基础服务）
builder.Services.AddControllersWithViews();

var app = builder.Build();

// 中间件管道配置
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

**理解要点**：
- `WebApplication.CreateBuilder()` 自动配置日志、配置、DI 容器
- `AddControllersWithViews()` 注册 MVC 所需的所有服务
- Controller 通过构造函数自动接收注入的服务

**关联教程**：[[基础篇/01-依赖注入详解]]

---

### 8. 中间件管道与请求处理 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

每个 HTTP 请求都会经过一系列中间件的处理：

**默认中间件管道**：
```
请求进入
    ↓
[异常处理中间件] ← HSTS 中间件 ← HTTPS 重定向中间件 ← 静态文件中间件 ← 路由中间件 ← 授权中间件 ← 终端中间件（MVC）
    ↓
响应返回
```

**Program.cs 配置**：
```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

**关键概念**：
- 中间件的执行顺序很重要（注册顺序 = 执行顺序）
- `Use` 方法可以短路管道（不调用 `next()`）
- 终端中间件不再调用 `next()`，直接生成响应

**关联教程**：[[基础篇/02-中间件管道]]

---

## 扩展练习

完成基础项目后，尝试以下 **5 个进阶任务** 来巩固所学知识：

### 🎯 练习 1：添加联系表单（难度：⭐⭐）
**目标**：在首页底部添加一个联系表单，包含姓名、邮箱、消息内容字段。

**要求**：
- 创建 `ContactViewModel` 数据模型，使用 Data Annotations 进行验证
- 实现 `HomeController.Contact()` POST Action 处理表单提交
- 添加客户端验证（jQuery Validation 或原生 HTML5 验证）
- 表单提交后显示成功提示（TempData 传递消息）

**涉及知识点**：模型绑定、表单验证、HTTP POST 请求、 TempData

**参考教程**：[[基础篇/03-表单处理与验证]]

---

### 🎯 练习 2：实现暗黑模式切换（难度：⭐⭐）
**目标**：允许用户在亮色/暗色主题之间切换，偏好设置保存到 LocalStorage。

**要求**：
- 定义 CSS 变量控制颜色主题
- 添加切换按钮到导航栏
- 使用 JavaScript 读写 LocalStorage
- 页面加载时根据保存的主题应用样式

**涉及知识点**：CSS 变量、JavaScript DOM 操作、浏览器存储 API

**参考教程**：[[进阶篇/08-前端工程化#主题切换]]

---

### 🎯 练习 3：添加作品集展示页（难度：⭐⭐⭐）
**目标**：新建 `PortfolioController` 和对应的视图，展示项目作品列表。

**要求**：
- 创建 `Project` 模型（标题、描述、截图、技术栈标签、GitHub 链接）
- 使用数据库或 JSON 文件存储项目数据（推荐先用 JSON）
- 实现详情页点击跳转
- 添加筛选功能（按技术栈分类过滤）

**涉及知识点**：新 Controller 创建、多视图路由、数据源抽象、Tag Helper

**参考教程**：[[基础篇/05-路由系统详解]]

---

### 🎯 练习 4：集成简单搜索功能（难度：⭐⭐⭐）
**目标**：在导航栏添加搜索框，支持对站内内容进行关键词搜索。

**要求**：
- 创建 `SearchController` 和 `Results` 视图
- 实现简单的字符串匹配算法（标题 + 内容搜索）
- 高亮显示搜索关键词
- 显示搜索结果数量统计

**涉及知识点**：查询参数传递、字符串操作、HTML 编码防 XSS、分页基础

**参考教程**：[[进阶篇/07-全文搜索实现]]

---

### 🎯 练习 5：Docker 化部署（难度：⭐⭐⭐⭐）
**目标**：将项目打包为 Docker 镜像，并在容器中运行。

**要求**：
- 编写 `Dockerfile`（多阶段构建：编译 → 运行）
- 创建 `docker-compose.yml`（简化启动命令）
- 配置健康检查端点
- 推送到 Docker Hub 或私有仓库
- 编写部署文档（环境变量、持久化卷、网络配置）

**涉及知识点**：Docker 基础、镜像分层、容器编排、CI/CD 入门

**参考教程**：[[高手篇/03-Docker容器化部署]]

---

## 常见问题 FAQ

### Q1：运行时提示端口被占用怎么办？
**A**：修改 `Properties/launchSettings.json` 中的 `applicationUrl`，或使用 `dotnet run --urls "http://localhost:5050"` 指定其他端口。

### Q2：图片无法显示（404 错误）？
**A**：确保图片放在 `wwwroot/images/` 目录下，引用路径使用 `~/images/xxx.jpg` 格式，且文件名大小写正确（Linux 区分大小写）。

### Q3：Razor 语法报错 "@ 符号无法识别"？
**A**：确认文件扩展名为 `.cshtml`（不是 `.html`），并且已安装 ASP.NET Core 工作负载（Visual Studio Installer → 修改 → 勾选"ASP.NET 和 Web 开发"）。

### Q4：Bootstrap 样式没有生效？
**A**：检查 CSS 文件路径是否正确（浏览器 F12 → Console 查看 404 错误），确认 `_Layout.cshtml` 中正确引入了 Bootstrap CDN 或本地文件。

### Q5：如何将项目上传到 GitHub？
**A**：
```bash
git init
git add .
git commit -m "Initial commit: Hello World project"
git remote add origin https://github.com/yourusername/hello-world.git
git push -u origin main
```
记得在项目根目录添加 `.gitignore` 文件（排除 `bin/`、`obj/`、`node_modules/` 等）。

---

## 关联教程索引

本项目的每个知识点都对应知识库中的详细教程：

| 知识点 | 教程位置 | 学习优先级 |
|--------|----------|------------|
| MVC 基础 | [[入门篇/01-MVC基础概念]] | 必学 |
| Razor 语法 | [[入门篇/02-Razor语法入门]] | 必学 |
| 第一个应用 | [[入门篇/03-第一个应用]] | 必学（本文档） |
| Layout 与 PartialView | [[入门篇/04-Layout与PartialView]] | 必学 |
| 静态文件 | [[入门篇/05-静态文件处理]] | 推荐 |
| Bootstrap 快速上手 | [[入门篇/06-Bootstrap快速上手]] | 推荐 |
| 依赖注入 | [[基础篇/01-依赖注入详解]] | 进阶必学 |
| 中间件管道 | [[基础篇/02-中间件管道]] | 进阶必学 |
| 表单验证 | [[基础篇/03-表单处理与验证]] | 进阶推荐 |
| 路由系统 | [[基础篇/05-路由系统详解]] | 进阶推荐 |
| Docker 部署 | [[高手篇/03-Docker容器化部署]] | 高级选学 |

---

## 下一步学习路线

完成本项目后，你已经具备了 ASP.NET Core 开发的基础能力。接下来建议按以下路径继续深入学习：

### 📚 推荐学习顺序

1. **[[todo-app/README.md]]** — Todo 待办事项管理应用
   - 学习 CRUD 操作、Entity Framework Core、依赖注入实战
   - 预计耗时：5-8 小时
   - 难度：⭐⭐⭐ 中级

2. **[[blog-system/README.md]]** — 博客系统
   - 学习 RESTful API 设计、JWT 认证、富文本编辑
   - 预计耗时：15-20 小时
   - 难度：⭐⭐⭐⭐ 高级

3. **[[ecommerce-mall/README.md]]** — CloudMall 微服务电商商城
   - 学习微服务架构、分布式事务、容器编排
   - 预计耗时：30-40 小时
   - 难度：⭐⭐⭐⭐⭐ 专家级

---

## 项目统计

- **总代码量**：~800 行（C# + Razor + CSS/JS）
- **文件数量**：25+ 个文件
- **涉及技术点**：8 个核心知识点
- **预计完成时间**：2-3 小时（初学者）
- **前置知识要求**：C# 基础语法、HTML/CSS 基础

---

## 许可证

本项目仅用于学习和教育目的。代码基于 MIT 许可证开源。

---

**最后更新时间**：2026-04-17
**维护者**：ASP.NET Core 知识库团队
**反馈渠道**：如有问题或建议，欢迎提交 Issue 或 Pull Request
