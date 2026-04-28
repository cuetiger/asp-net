# VS Code 必装扩展清单

> 扩展是 VS Code 强大功能的源泉。本文为 ASP.NET 开发者精心挑选了 50+ 个实用扩展，涵盖 C# 开发、Web 前端、Git 集成、代码质量、效率提升、容器化和数据库等各个方面，帮助你打造高效开发环境。

---

## 目录

- [一、C# 开发核心扩展](#一c-开发核心扩展)
- [二、Web 开发扩展](#二web-开发扩展)
- [三、Git 集成扩展](#三git-集成扩展)
- [四、代码质量扩展](#四代码质量扩展)
- [五、效率提升扩展](#五效率提升扩展)
- [六、Docker/Kubernetes 扩展](六dockerkubernetes-扩展)
- [七、数据库扩展](#七数据库扩展)
- [八、扩展冲突排查与性能优化](#八扩展冲突排查与性能优化)
- [九、推荐的工作区 settings.json 配置](#九推荐的工作区-settingsjson-配置)

---

## 一、C# 开发核心扩展

作为 ASP.NET 开发者，以下扩展是构建高效 C# 开发环境的基石。

### 1. C# Dev Kit

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-dotnettools.csdevkit` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-dotnettools.csdevkit` |
| **下载量** | 10M+ |

**功能描述**：

C# Dev Kit 是微软官方推出的 C# 开发工具包，它将 Visual Studio 中强大的 C# 开发体验带入 VS Code。这是 ASP.NET Core 开发的**必备扩展**。

**核心功能**：
- 完整的 IntelliSense 智能代码补全
- 高级重构功能（提取方法、重命名、接口实现等）
- 解决方案资源管理器（Solution Explorer）
- 单元测试运行与调试
- NuGet 包管理 GUI
- 代码导航（定义跳转、引用查找、类型层次结构）
- 内置终端中的 .NET CLI 支持
- 调试支持（启动配置自动生成）

**适用场景**：所有 .NET/C# 项目开发

```
依赖关系说明：
C# Dev Kit 会自动安装以下组件：
├── C# (ms-dotnettools.csharp)          # C# 语言基础支持
├── .NET Install Tool                   # .NET SDK 安装管理
├── IntelliCode for C#                  # AI 辅助编程
├── Debugger for .NET                   # .NET 调试器
└── C# Extensions (jchannon.csharpextensions) # 额外 C# 功能
```

### 2. C# IntelliCode

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-dotnettools.vscodeintellicode` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-dotnettools.vscodeintellicode` |

**功能描述**：

基于 AI 和机器学习的智能代码补全工具。通过分析数千个开源 GitHub 项目，IntelliCode 能够预测你最可能需要的代码，并将最佳建议置于列表顶部。

**核心功能**：
- **整行代码补全**：根据上下文预测并建议完整的一行代码
- **API 使用示例**：基于真实项目提供 API 用法示例
- **参数推断**：智能预测方法调用的参数值
- **自定义模型训练**：可使用自己的代码库训练个性化模型

```csharp
// IntelliCode 整行补全示例：
// 输入: var users = await _context.Users
// IntelliCode 建议: .Where(u => u.IsActive).ToListAsync();

// API 示例提示：
// 输入: HttpClient.
// 显示: GetAsync, PostAsync, PutAsync, DeleteAsync 等
// 并附带常见用法示例
```

**适用场景**：希望提高编码速度和代码质量的开发者

### 3. C# (基础语言支持)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-dotnettools.csharp` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-dotnettools.csharp` |

**功能描述**：

C# 语言的基础 OmniSharp 分析引擎，提供语法高亮、基础 IntelliSense、错误诊断等功能。通常作为 C# Dev Kit 的依赖自动安装。

**核心功能**：
- 基于 Roslyn 的语义分析
- 语法着色和高亮显示
- 基础代码补全
- 实时错误检测
- 支持多目标框架（.NET Framework / .NET Core / .NET 5+）

### 4. C# Extensions (jchannon)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `jchannon.csharpextensions` |
| **发布者** | Jason Channon |
| **安装命令** | `ext install jchannon.csharpextensions` |

**功能描述**：

为 C# 开发添加额外的便捷功能和代码片段集合。

**核心功能**：
- 快速创建类/接口/枚举/结构体的文件模板
- 常用代码片段（属性、方法、构造函数等）
- 区域折叠标记生成
- JSON 配置转换
- 排序和整理 using 语句

**常用代码片段**：

```
触发词        生成的代码模板
─────────────────────────────────────
prop         public int MyProperty { get; set; }
propfull     private int myProperty; public int MyProperty { get => myProperty; set => myProperty = value; }
propg        public int MyProperty { get; private set; }
ctor         构造函数模板
class        类定义模板
interface    接口定义模板
enum         枚举定义模板
try          try-catch 模板
for          for 循环模板
foreach      foreach 循环模板
if           if 条件判断模板
```

### 5. NSwag: OpenAPI/Swagger

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `rickeverett.nswag-vscode` |
| **发布者** | Ric Everett |
| **安装命令** | `ext install rickeverett.nswag-vscode` |

**功能描述**：

在 VS Code 中直接编辑 NSwag 配置文件，用于从 OpenAPI/Swagger 规范生成 C# 客户端代码。

**核心功能**：
- NSwag 配置文件的语法高亮和验证
- 从 Swagger/OpenAPI JSON/YAML 生成 C# TypeScript Client
- 一键生成 API 客户端代码
- 支持自定义生成设置

**适用场景**：前后端分离项目中生成强类型的 API 客户端

```yaml
# nswag.json 配置示例
{
  "runtime": "NetCore60",
  "defaultEnumMode": "StringEnum",
  "documentGenerator": {
    "fromDocument": {
      "url": "https://localhost:5001/swagger/v1/swagger.json"
    }
  },
  "codeGenerators": {
    "openApiToCSharpClient": {
      "className": "{Controller}Client",
      "output": "./Generated/{Controller}Client.cs",
      "namespace": "MyApp.ApiClients"
    }
  }
}
```

### 6. .NET Core Task Runner

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `formulahendry.dotnet` |
| **发布者** | Jun Han |
| **安装命令** | `ext install formulahendry.dotnet` |

**功能描述**：

在侧边栏中提供 .NET Core CLI 的图形化操作界面。

**核心功能**：
- 可视化的解决方案/项目管理
- 一键执行 build/test/run/publish
- NuGet 包的搜索和管理
- 项目依赖关系可视化
- 快速创建新项目/文件

### 7. Auto-Using for C#

| 属性 | 信息 |
|------|------|
| **扩展 ID** `k--kato.docomment` |
| **发布者** | Kazuki Kato |
| **安装命令** | `ext install k--kato.docomment` |

**功能描述**：

自动为 C# 类型添加缺失的 using 引用语句。

**核心功能**：
- 输入未引用的类型名时自动添加 using
- 删除不再使用的 using 语句
- 对 using 语句进行排序
- 支持自定义命名空间别名

### 8. Solution Explorer

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `fernandoescolar.vscode-solution-explorer` |
| **发布者** | Fernando Escolar |
| **安装命令** | `ext install fernandoescolar.vscode-solution-explorer` |

**功能描述**：

类似 Visual Studio 的解决方案资源管理器，以解决方案结构展示项目文件。

**核心功能**：
- 以 .sln 文件为中心的项目视图
- 支持多层嵌套的项目结构
- 右键菜单支持新建/删除/重命名文件
- 项目依赖关系树状图
- 筛选和搜索项目项

### 9. Code Metrics

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `otykzt.code-metrics` |
| **发布者** | otykzt |
| **安装命令** | `ext install otykzt.code-metrics` |

**功能描述**：

计算和显示 C# 代码的复杂度指标。

**核心功能**：
- 圈复杂度（Cyclomatic Complexity）计算
- 继承深度（Depth of Inheritance）
- 类耦合度（Class Coupling）
- 代码行数统计
- 维护性指数（Maintainability Index）

### 10. Roslynator

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `josefpihrt-vscode.roslynator` |
| **发布者** | Josef Pihrt |
| **安装命令** | `ext install josefpihrt-vscode.roslynator` |

**功能描述**：

基于 Roslyn 的 C# 代码分析和重构工具集。

**核心功能**：
- 200+ 个代码修复和分析器
- 大量额外的代码重构选项
- 代码格式化增强
- 性能优化建议

---

## 二、Web 开发扩展

ASP.NET 开发不可避免地涉及前端工作，这些扩展能显著提升 Web 开发体验。

### 11. HTML CSS Support

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ecmel.vscode-html-css` |
| **发布者** | Ecmel Ercan |
| **安装命令** | `ext install ecmel.vscode-html-css` |

**功能描述**：

增强 HTML/CSS 的智能感知能力，支持 CSS 类名补全和颜色预览。

**核心功能**：
- CSS 类名自动补全
- HTML 标签属性智能提示
- CSS 颜色值的可视化预览
- 支持 SCSS/Less/Stylus
- 从 CSS 文件中提取类名用于 HTML

```html
<!-- 类名补全示例 -->
<div class="container">  <!-- 自动补全 container -->
    <button class="btn btn-primary">
        <!-- 提示所有可用的 btn-* 类 -->
    </button>
</div>
```

### 12. Live Server

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ritwickdey.liveserver` |
| **发布者** | Ritwick Dey |
| **安装命令** | `ext install ritwickdey.liveserver` |

**功能描述**：

启动本地开发服务器，具有实时刷新功能。对于 Blazor WebAssembly 或纯前端项目的开发非常有用。

**核心功能**：
- 一键启动本地 HTTP 服务器
- 文件保存后自动刷新浏览器
- 支持静态资源和 SPA 路由
- 自定义端口和根目录
- HTTPS 支持

**适用场景**：Blazor WASM 前端开发、静态网站预览

### 13. Auto Rename Tag

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `formulahendry.auto-rename-tag` |
| **发布者** | Jun Han |
| **安装命令** | `ext install formulahendry.auto-rename-tag` |

**功能描述**：

修改 HTML/XML 标签时自动同步修改配对标签。

**核心功能**：
- 编辑开始标签时自动更新结束标签
- 编辑结束标签时自动更新开始标签
- 支持 HTML、JSX、Vue、Angular 等模板语法
- 支持 Razor 视图

```html
<!-- 修改前 -->
<div class="old-class"></div>

<!-- 将 div 改为 section 后 -->
<section class="old-class"></section>  <!-- 结束标签自动更新 -->
```

### 14. Path Intellisense

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `christian-kohler.path-intellisense` |
| **发布者** | Christian Kohler |
| **安装命令** | `ext install christian-kohler.path-intellisense` |

**功能描述**：

自动补全文件路径，避免手动输入路径的错误。

**核心功能**：
- 文件路径自动完成
- 相对路径和绝对路径支持
- 支持 src/href/url 等多种属性
- 自动识别当前工作目录

```razor
<!-- 在 Razor 视图中 -->
<img src="./images/logo.png" />  <!-- 自动补全路径 -->

<link rel="stylesheet" href="~/css/site.css" />
```

### 15. Thunder Client

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `rangav.thunder-client` |
| **发布者** | Ranga Vaziran |
| **安装命令** | `ext install rangav.thunder-client` |

**功能描述**：

轻量级的 REST API 测试客户端，类似于 Postman 但完全集成在 VS Code 中。

**核心功能**：
- 发送 GET/POST/PUT/DELETE/PATCH 请求
- 环境变量管理
- 请求历史记录
- 集合（Collection）组织
- 代码生成（cURL/各种语言）
- 响应格式化和高亮显示
- 认证支持（Bearer Token/OAuth2/Basic Auth）

```http
### Thunder Client 请求示例 (.thunder 或 .http 文件)

@base_url = http://localhost:5000/api

### 获取用户列表
GET {{base_url}}/users
Authorization: Bearer {{token}}
Content-Type: application/json

### 创建用户
POST {{base_url}}/users
Content-Type: application/json

{
  "name": "张三",
  "email": "zhangsan@example.com"
}
```

### 16. JavaScript (ES6) Code Snippets

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `xabikos.javascriptsnippets` |
| **发布者** | Charalampos Karypidis |
| **安装命令** | `ext install xabikos.javascriptsnippets` |

**功能描述**：

JavaScript ES6+ 的代码片段集合，覆盖日常开发的常用模式。

**常用代码片段**：

```
触发词       生成内容
─────────────────────────────────
imp→        import moduleName from 'module'
imn→        import 'module'
imd→        import { destructuredModule } from 'module'
cla→       class name { constructor() {} }
clae→      export default class name { constructor() {} }
fo→        for (let i = 0; i < length; i++) {}
fof→       for (let item of object) {}
fn→        const add = (params) => {}
fe→        fetch('url', { method: 'GET' })
then→      .then(res => res.json())
async→     const methodName = async (params) => => {}
prom→      return new Promise((resolve, reject) => {})
```

### 17. ESLint

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `dbaeumer.vscode-eslint` |
| **发布者** | Dirk Baeumer |
| **安装命令** | `ext install dbaeumer.vscode-eslint` |

**功能描述**：

集成 ESLint JavaScript/TypeScript 代码检查工具，实时发现代码问题。

**核心功能**：
- 实时 lint 错误和警告显示
- 保存时自动修复问题
- 与 prettier 协同工作
- 自定义规则配置支持

### 18. Prettier - Code Formatter

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `esbenp.prettier-vscode` |
| **发布者** | Prettier |
| **安装命令** | `ext install esbenp.prettier-vscode` |

**功能描述**：

使用 Prettier 格式化 JavaScript/TypeScript/HTML/CSS/JSON/Markdown 等多种语言。

**核心功能**：
- 保存时自动格式化 (`Shift+Alt+F`)
- 统一代码风格
- 支持自定义配置文件 (.prettierrc)
- 与 ESLint 无缝配合

---

## 三、Git 集成扩展

VS Code 内置了基础的 Git 支持，但以下扩展能让 Git 工作流更加流畅。

### 19. GitLens — Git supercharged

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `eamodio.gitlens` |
| **发布者** | GitKraken |
| **安装命令** | `ext install eamodio.gitlens` |
| **下载量** | 25M+ |

**功能描述**：

VS Code 最强大的 Git 扩展，被誉为"Git 的超级增强"。对于任何使用 Git 的开发者来说都是必装扩展。

**核心功能**：
- **行级 Git 注释**：每行代码旁边显示最后修改者和提交信息
- ** blame 视图**：查看整个文件的修改历史
- **提交历史时间线**：可视化的提交记录
- **作者热度图**：了解谁修改了哪些代码
- **比较变更**：详细的 diff 视图
- **搜索提交历史**：按内容/作者/日期搜索
- **仓库状态概览**：一目了然的分支和工作区状态
- **远程仓库链接**：直接跳转到 GitHub/GitLab 的对应位置

**GitLens 注释效果示例**：

```csharp
public class UserService  // ← 12 天前 by 张三 in feat/add-user-service
{
    private readonly IUserRepository _repository;
    // ↑ 5 天前 by 李四 in refactor/use-dependency-injection

    public async Task<User?> GetByIdAsync(int id)
    // ↑ 今天 by 你 in fix/user-not-found-bug
    {
        return await _repository.GetByIdAsync(id);
    }
}
```

**适用场景**：所有使用 Git 版本控制的项目

### 20. Git Graph

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `mhutchie.git-graph` |
| **发布者** | mhutchie |
| **安装命令** | `ext install mhutchie.git-graph` |

**功能描述**：

可视化的 Git 分支图表，直观地查看分支结构和合并历史。

**核心功能**：
- 图形化分支视图
- 拖拽式合并和变基操作
- cherry-pick 操作
- 创建/删除/切换分支
- 提交详情查看
- 标签管理

**适用场景**：需要理解复杂分支历史的团队项目

### 21. Git History

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `donjayamanne.githistory` |
| **发布者** | Don Jayamanne |
| **安装命令** | `ext install donjayamanne.githistory` |

**功能描述**：

查看文件或仓库的 Git 历史、比较分支差异。

**核心功能**：
- 文件修改历史
- 作者搜索
- 时间范围筛选
- 行级别历史（特定行的修改记录）
- 分支对比
- 撤销提交到指定版本

### 22. Git Blame

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `wadowskiki.git-blame` |
| **发布者** | waderyan |
| **安装命令** | `ext install wadowskiki.git-blame` |

**功能描述**：

内联显示每行代码的 Git blame 信息（谁在什么时候修改了这行代码）。

**核心功能**：
- 行级 blame 信息显示
- 点击跳转到原始提交
- 打开完整的 blame 视图
- 可配置的信息显示格式

### 23. GitLens Insiders (可选)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `eamodio.gitlens-insiders` |
| **发布者** | GitKraken |
| **安装命令** | `ext install eamodio.gitlens-insiders` |

**功能描述**：

GitLens 的高级版，提供 Worktrees 视图、GitHub/GitLab/Azure DevOps PR 集成等企业级功能。

**核心功能**：
- Pull Requests 和 Issues 集成
- GitHub Actions 工作流可视化
- 多仓库工作区支持
- 高级搜索和过滤
- 团队协作分析

### 24. Badges (Git 状态徽章)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `badges.badges` |
| **发布者** | Badges |
| **安装命令** | `ext install badges.badges` |

**功能描述**：

在状态栏显示 Git 分支名称、远程同步状态等信息。

---

## 四、代码质量扩展

保持代码质量是专业开发者的标志，这些扩展帮助你在编写时就发现问题。

### 25. Error Lens

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `usernamehw.errorlens` |
| **发布者** | Alexander |
| **安装命令** | `ext install usernamehw.errorlens` |

**功能描述**：

在代码行内直接显示编译错误和警告信息，无需悬停或查看问题面板。

**效果演示**：

```csharp
// 普通模式：需要鼠标悬停才能看到错误
var result = await GetUser(id);  // ← 有红色波浪线，但不知道什么错

// Error Lens 模式：直接在行内显示
var result = await GetUser(id); // CS0103 当前上下文中不存在名称"GetUser"
                               // CS7036 缺少对应于所需形参"id"的实参
```

**核心功能**：
- 行内显示诊断信息（错误/警告/提示/信息）
- 错误严重程度颜色区分
- 可配置显示哪些级别的诊断
- 支持 C#/TypeScript/Python 等多种语言

**适用场景**：快速发现和定位代码问题

### 26. Code Spell Checker

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `streetsidesoftware.code-spell-checker` |
| **发布者** | Street Side Software |
| **安装命令** | `ext install streetsidesoftware.code-spell-checker` |

**功能描述**：

源代码拼写检查器，捕捉注释、字符串和标识符中的拼写错误。

**核心功能**：
- 实时拼写检查
- 支持驼峰命名和下划线命名的单词识别
- 用户词典自定义
- 多语言支持
- 特定语言的字典扩展（如中文拼音）

```csharp
// 拼写检查示例
/// <summary>
/// Retreives user by identifiyer  // ← Unknown words: Retreives, identifiyer
/// </summary>
public User? GetUserById(int userId)  // OK - 已知单词
{
    // ...
}

// 自定义词典：cspell.json
{
  "words": [
    "AspNet",
    "DbContext",
    "middleware"
  ]
}
```

### 27. Trailing Spaces

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `shardulm94.trailing-spaces` |
| **发布者** | Shardul Mahajan |
| **安装命令** | `ext install shardulm94.trailing-spaces` |

**功能描述**：

高亮显示并允许一键删除行尾空白字符。

**核心功能**：
- 行尾空格高亮显示
- 保存时自动删除尾随空格
- 状态栏显示当前文件的尾随空格数量
- 支持仅高亮不删除的模式

### 28. Bracket Pair Colorizer 2

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `coenraads.bracket-pair-colorizer-2` |
| **发布者** | CoenraadS |
| **安装命令** | `ext install coenraads.bracket-pair-colorizer-2` |

**功能描述**：

用不同颜色匹配括号对，使嵌套结构一目了然。

**效果演示**：

```csharp
// 不同颜色的括号配对
public async Task<IActionResult> Action(
    [FromBody] RequestDto request)  // 红色 ()
{                                    // 绿色 {
    var result = await _service      // 蓝色 (
        .ProcessAsync(               // 黄色 (
            request.Data            // 紫色 [
        )                            // 黄色 )
    );                               // 蓝色 )

    return Ok(result);               // 绿色 }
}                                    // 红色 )
```

> **注意**: VS Code 1.60+ 已内置括号着色功能，此扩展可作为增强替代方案。

### 29. DotENV

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `mikestead.dotenv` |
| **发布者** | Mike Stead |
| **安装命令** | `ext install mikestead.dotenv` |

**功能描述**：

为 `.env` 文件提供语法高亮和自动补全。

**核心功能**：
- .env 文件语法高亮
- 多环境文件支持（.env.development, .env.production）
- 自动补全已定义的环境变量
- 键值对验证

```bash
# .env.development
ConnectionStrings__Default=Server=localhost;Database=MyApp_Dev;...
ASPNETCORE_ENVIRONMENT=Development
JWT__Secret=my-development-secret-key
JWT__Issuer=https://localhost:5001
```

### 30. XML Tools

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `dotjoshjohnson.xml` |
| **发布者** | Josh Johnson |
| **安装命令** | `ext install dotjoshjohnson.xml` |

**功能描述**：

XML 文件的格式化、验证和转换工具。

**核心功能**：
- XML 格式化
- XML 验证（XSD/DTD）
- XPath 求值
- XML <-> JSON 转换
- 注释/取消注释

**适用场景**：处理 appsettings.json、.csproj、.pubxml 等 XML 配置文件

### 31. EditorConfig for VS Code

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `editorconfig.editorconfig` |
| **发布者** | EditorConfig |
| **安装命令** | `ext install editorconfig.editorconfig` |

**功能描述**：

支持 EditorConfig 配置文件，确保团队代码风格一致。

```ini
# .editorconfig 示例
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[*.{json,yml,yaml}]
indent_size = 2
```

### 32. Markdown All in One

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `yzhang.markdown-all-in-one` |
| **发布者** | Yu Zhang |
| **安装命令** | `ext install yzhang.markdown-all-in-one` |

**功能描述**：

全面的 Markdown 编辑增强工具。

**核心功能**：
- 快捷键格式化（加粗/斜体/标题/列表等）
- 目录（TOC）自动生成
- 数学公式支持
- 表格格式化
- 图片粘贴路径自动处理
- 导出为 PDF/HTML

---

## 五、效率提升扩展

这些扩展虽然不是必需品，但能显著改善开发体验和效率。

### 33. Material Icon Theme

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `PKief.material-icon-theme` |
| **发布者** | Philipp Kief |
| **安装命令** | `ext install PKief.material-icon-theme` |
| **下载量** | 20M+ |

**功能描述**：

最流行的文件图标主题，为不同类型的文件赋予独特的图标，让文件浏览更直观。

**支持的图标类别**：
- 编程语言（C#, TypeScript, Python, Go, Rust...）
- 框架（.NET, Angular, React, Vue...）
- 配置文件（JSON, YAML, XML, .env...）
- 数据库（SQL, SQLite, Redis...）
- 云服务（Docker, K8s, Azure, AWS...）
- 文档（MD, PDF, TXT...）

**适用场景**：所有项目，强烈推荐安装

### 34. One Dark Pro

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `zhuangtongfa.material-theme` |
| **发布者** | zhuangtongfa |
| **安装命令** | `ext install zhuangtongfa.material-theme` |
| **下载量** | 15M+ |

**功能描述**：

Atom 编辑器 One Dark 主题的 VS Code 移植版，最受欢迎的深色主题之一。

**特点**：
- 护眼的深色调配色
- 对比度适中，长时间编码不易疲劳
- 支持 Material 主题图标配套
- 多种变体（Bold/Darker/One Light）

### 35. Project Manager

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `alefragnani.project-manager` |
| **发布者** | Alessandro Fragnani |
| **安装命令** | `ext install alefragnani.project-manager` |

**功能描述**：

方便地在多个项目/工作区间快速切换。

**核心功能**：
- 保存和恢复项目列表
- 项目分组管理
- 最近使用的项目自动记录
- 自动打开相关终端和文件

**适用场景**：同时维护多个项目的开发者

### 36. Settings Sync

| 属性 | 信息 |
|------|------|
| **扩展 ID** | (内置) |  |
| **说明** | VS Code 内置功能 |  |

**功能描述**：

使用 GitHub 或 Microsoft 账户同步 VS Code 设置、快捷键绑定、扩展和代码片段。

**使用方法**：
1. `Ctrl+Shift+P` -> "Settings Sync: Turn On"
2. 登录 GitHub/Microsoft 账户
3. 选择要同步的内容（Settings/Keybindings/Extensions/Snippets/Tasks）
4. 在其他设备上登录即可恢复配置

**同步内容**：
```
├── User Settings (settings.json)
├── Keyboard Shortcuts (keybindings.json)
├── Installed Extensions + Extension Settings
├── Code Snippets
├── Tasks (tasks.json)
└── UI State (布局/主题/图标)
```

### 37. TODO Highlight

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `wayou.todo-tree` |
| **发布者** | Gruntfuttuck |
| **安装命令** | `ext install wayou.todo-tree` |

**功能描述**：

高亮显示代码中的 TODO、FIXME、HACK 等注释标记，并在侧边栏生成待办事项树。

**支持的默认关键词**：
- `TODO:` - 待办事项
- `FIXME:` - 需要修复的问题
- `BUG:` - 已知的 Bug
- `HACK:` - 临时解决方案
- `NOTE:` - 重要备注
- `REVIEW:` - 需要审查的代码

```csharp
public class PaymentService
{
    // TODO: 实现支付回调通知处理
    // FIXME: 金额计算存在精度丢失问题
    // HACK: 临时绕过第三方API限制
    // BUG: 并发情况下可能出现重复支付
    // NOTE: 此方法需要优化性能
    // REVIEW: 安全性审计待确认
    
    public async Task ProcessPayment(PaymentRequest request)
    {
        // ...
    }
}
```

### 38. Toggle Quotes

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `bceskav.toggle-quotes` |
| **发布者** | bceskav |
| **安装命令** | `ext install bceskav.toggle-quotes` |

**功能描述**：

快速在单引号、双引号、反引号之间切换字符串引号类型。

**核心功能**：
- 光标在引号内时按快捷键切换引号类型
- 支持 `'` -> `"` -> `` ` `` 循环切换
- 同时切换配对的引号

### 39. Regex Previewer

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `chrmarti.regex` |
| **发布者** | Christoph Martens |
| **安装命令** | `ext install chrmarti.regex` |

**功能描述**：

实时预览正则表达式匹配结果。

**核心功能**：
- 正则表达式语法高亮
- 实时匹配预览
- 匹配组高亮显示
- 支持常见的正则表达式风格

### 40. Indent Rainbow

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `oderwat.indent-rainbow` |
| **发布者** | oderwat |
| **安装命令** | `ext install oderwat.indent-rainbow` |

**功能描述**：

用不同颜色标注不同的缩进层级，便于识别代码块结构。

**适用场景**：阅读复杂的嵌套代码时特别有用

---

## 六、Docker/Kubernetes 扩展

现代 ASP.NET 应用越来越依赖容器化部署，这些扩展简化了容器开发流程。

### 41. Docker

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-azuretools.vscode-docker` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-azuretools.vscode-docker` |
| **下载量** | 20M+ |

**功能描述**：

微软官方 Docker 扩展，提供完整的容器开发支持。

**核心功能**：
- Dockerfile 语法高亮和智能提示
- docker-compose.yml 验证和补全
- 容器镜像管理（构建/推送/拉取）
- 容器生命周期管理（启动/停止/重启/删除）
- 容器日志查看
- 容器内部终端访问
- Dockerfile 代码片段

```dockerfile
# Dockerfile 代码片段示例
# 输入: dockerfile → 选择 aspnet 模板

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"
COPY . .
WORKDIR "/src/MyApp"
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### 42. Kubernetes

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-kubernetes-tools.vscode-kubernetes-tools` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-kubernetes-tools.vscode-kubernetes-tools` |

**功能描述**：

Kubernetes 配置文件管理和集群交互工具。

**核心功能**：
- YAML 清单文件语法高亮和验证
- Helm Chart 模板支持
- 集群资源浏览器
- Pod 日志查看和 Shell 访问
- 应用部署和扩缩容
- CRD（自定义资源）支持

### 43. YAML

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `redhat.vscode-yaml` |
| **发布者** | Red Hat |
| **安装命令** | `ext install redhat.vscode-yaml` |

**功能描述**：

YAML 语言支持增强，包括语法验证、自动补全和格式化。

**核心功能**：
- YAML 语法验证和错误检测
- Schema 自动补全（K8s、Azure Pipelines、OpenAPI 等）
- 自定义 Schema 关联
- 格式化和排序支持

### 44. Remote - SSH

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-vscode-remote.remote-ssh` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-vscode-remote.remote-ssh` |

**功能描述**：

通过 SSH 连接远程服务器进行开发，就像在本地一样。

**核心功能**：
- SSH 连接远程 Linux 服务器
- 远程文件系统浏览和编辑
- 远程终端访问
- 远程扩展安装
- 远程端口转发
- 远程调试支持

**适用场景**：在云服务器或 Docker 容器中进行开发

### 45. Remote - Containers

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-vscode-remote.remote-containers` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-vscode-remote.remote-containers` |

**功能描述**：

在容器内进行完整的开发环境搭建和使用。

**核心功能**：
- 使用 devcontainer.json 定义开发容器
- 一键创建/重建开发容器
- 容器内安装扩展
- 容器内运行和调试
- 共享卷挂载

```json
// .devcontainer/devcontainer.json 示例
{
  "name": "ASP.NET Core 8.0",
  "image": "mcr.microsoft.com/devcontainers/dotnet:8.0",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/azure-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csdevkit",
        "ms-azuretools.vscode-docker"
      ]
    }
  },
  "forwardPorts": [5000, 5001]
}
```

---

## 七、数据库扩展

数据库操作是后端开发的重要组成部分，这些扩展让你不用离开 VS Code 就能管理数据库。

### 46. SQL Server (mssql)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `ms-mssql.mssql` |
| **发布者** | Microsoft |
| **安装命令** | `ext install ms-mssql.mssql` |

**功能描述**：

Microsoft SQL Server 数据库的管理和查询工具。

**核心功能**：
- 连接 SQL Server / Azure SQL Database
- SQL 查询编辑器和执行
- 结果表格查看和导出
- 数据库对象浏览器（表/视图/存储过程/函数）
- 查询计划可视化
- 代码片段（CREATE TABLE/SELECT/INSERT 等）
- 连接 profile 管理

```sql
-- SQL Server 扩展代码片段示例
-- 输入: sqlSelect → 生成 SELECT 模板
SELECT TOP (1000) [Id]
      ,[Name]
      ,[Email]
      ,[CreatedAt]
  FROM [dbo].[Users]
  WHERE [IsActive] = 1
  ORDER BY [CreatedAt] DESC;
```

### 47. PostgreSQL

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `cjolowicz.postgres` |
| **发布者** | Christian Jolowicz |
| **安装命令** | `ext install cjolowicz.postgres` |

**功能描述**：

PostgreSQL 数据库管理扩展。

**核心功能**：
- PostgreSQL 连接管理
- 查询执行和结果查看
- 数据库结构浏览
- SQL 语法高亮
- EXPLAIN ANALYZE 支持

### 48. Redis

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `cweijan.vscode-redis` |
| **发布者** | cweijan |
| **安装命令** | `ext install cweijan.vscode-redis` |

**功能描述**：

Redis 缓存数据查看和管理工具。

**核心功能**：
- 连接 Redis 服务器（单机/集群/Sentinel）
- Key 浏览和搜索
- Value 查看（String/List/Set/Hash/ZSet）
- TTL 管理
- 数据导入导出
- 命令执行

**适用场景**：使用 Redis 作为分布式缓存的 ASP.NET 应用

### 49. SQLite Viewer

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `qwtel.sqlite-viewer` |
| **发布者** | qwtel |
| **安装命令** | `ext install qwtel.sqlite-viewer` |

**功能描述**：

SQLite 数据库文件的可视化查看器。

**核心功能**：
- 直接打开 .db/.sqlite/.sqlite3 文件
- 表数据浏览和编辑
- SQL 查询执行
- 表结构查看
- 数据导出（CSV/JSON）

**适用场景**：开发和测试中使用 SQLite 作为轻量级数据库

### 50. Database Client (通用)

| 属性 | 信息 |
|------|------|
| **扩展 ID** | `cweijan.dbclient-jdbc` |
| **发布者** | cweijan |
| **安装命令** | `ext install cweijan.dbclient-jdbc` |

**功能描述**：

通用的数据库客户端，支持 MySQL、Oracle、DB2、Derby 等多种数据库。

**核心功能**：
- 多种数据库连接支持
- 统一的查询界面
- 结果集查看和编辑
- 数据导出功能

---

## 八、扩展冲突排查与性能优化

随着扩展数量增加，可能会遇到冲突或性能问题。

### 常见问题及解决方案

#### 问题1：扩展导致 VS Code 启动变慢

```
诊断步骤：
1. 按 Ctrl+Shift+P -> "Developer: Show Running Extensions"
2. 查看每个扩展的激活时间和内存占用
3. 禁用不常用的扩展

优化策略：
- 只启用当前项目需要的扩展
- 使用扩展工作区配置（而非全局配置）
- 定期清理不再使用的扩展
- 关注扩展的激活事件（activationEvents）
```

#### 问题2：快捷键冲突

```
解决方法：
1. Ctrl+K Ctrl+S 打开快捷键设置
2. 搜索冲突的快捷键
3. 查看哪些扩展使用了相同的键绑定
4. 通过 keybindings.json 覆盖或禁用冲突的绑定

# 示例：解决 F2 冲突
[
  {
    "key": "f2",
    "command": "-editor.action.rename",  // 禁用原有
    "when": "editorTextFocus && !editorReadonly"
  },
  {
    "key": "f2",
    "command": "myCustomRename",         // 替换为自定义
    "when": "editorHasRenameProvider && editorTextFocus"
  }
]
```

#### 问题3：多个 Linting 扩展冲突

```
常见冲突组合：
- ESLint + TSLint（TSLint 已废弃，移除即可）
- C# Analyzer + ReSharper（选择其一）
- Prettier + EditorConfig formatter（统一配置）

解决原则：
1. 明确每种工具的职责边界
2. 对于相同功能只保留一个扩展
3. 合理配置各工具的协作方式
```

### 性能监控命令

```
# 查看扩展内存占用
Ctrl+Shift+P -> "Developer: Show Running Extensions"

# 查看进程信息
Ctrl+Shift+P -> "Developer: Process Info"

# 重载窗口（清除缓存）
Ctrl+Shift+P -> "Developer: Reload Window"

# 检查扩展健康状态
Ctrl+Shift+P -> "Extensions: Check for Missing Dependencies"
```

---

## 九、推荐的工作区 settings.json 配置

以下是针对 ASP.NET 开发优化的 VS Code 工作区配置，可以保存到 `.vscode/settings.json` 中供团队共享。

```json
{
  // ==================== 编辑器基本配置 ====================
  
  // 字体配置
  "editor.fontSize": 14,
  "editor.fontFamily": "'Cascadia Code', 'Fira Code', Consolas, 'Courier New'",
  "editor.fontLigatures": true,
  "editor.lineHeight": 1.6,
  
  // Tab 和缩进
  "editor.tabSize": 4,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false,
  
  // 自动保存
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000,
  
  // 格式化
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "csharp.vscode-csharp",
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "files.trimTrailingWhitespace": false
  },
  
  // 行号和 Minimap
  "editor.lineNumbers": "relative",
  "editor.minimap.enabled": true,
  "editor.minimap.maxColumn": 80,
  
  // Word Wrap
  "editor.wordWrap": "on",
  "editor.wordWrapColumn": 120,
  "editor.rulers": [120],
  
  // ==================== C# 特定配置 ====================
  
  // OmniSharp 设置
  "omnisharp.enableImportCompletion": true,
  "omnisharp.enableRoslynAnalyzers": true,
  "omnisharp.analyzeOpenDocumentsOnly": true,
  "omnisharp.useGlobalMono": "never",
  
  // 代码样式
  "csharp.format.enable": true,
  "[csharp]": {
    "editor.defaultFormatter": "csharp.vscode-csharp",
    "editor.suggest.selectionMode": "first",
    "editor.inlayHints.enabled": "offUnlessPressed"
  },
  
  // 分析器级别
  "dotnet.diagnosticSeverityRules": {
    "CS0067": "warning",   // 未使用的事件
    "CS0168": "warning",   // 未使用的变量
    "CS0219": "warning",   // 未使用的变量赋值
    "CS0414": "warning",   // 未使用的私有字段
    "CS0649": "suggestion", // 字段从未赋值
    "CS8618": "warning",   // 可空引用类型
    "CS8600": "warning",   // 可空转换
    "CS8602": "warning",   // 可能解引用空引用
    "CS8603": "warning",   // 可能返回空引用
    "CS8604": "warning"    // 可能传入空引用
  },
  
  // IntelliCode
  "vsintellicode.apiSurvey.models.csharp": ["*"],
  
  // ==================== 文件配置 ====================
  
  // 排除文件
  "files.exclude": {
    "**/bin": true,
    "**/obj": true,
    "**/node_modules": true,
    "**/.git": true,
    "**/*.dll": true,
    "**/*.pdb": true
  },
  
  // 搜索排除
  "search.exclude": {
    "**/bin": true,
    "**/obj": true,
    "**/node_modules": true,
    "**/.git": true
  },
  
  // 文件关联
  "files.associations": {
    "*.cshtml": "html",
    "*.razor": "html",
    "*.csproj": "xml",
    "*.config": "xml",
    "*.props": "xml",
    "*.targets": "xml",
    "*.nuspec": "xml",
    "*.globaljson": "json"
  },
  
  // 默认换行符
  "files.eol": "\n",
  
  // ==================== 终端配置 ====================
  
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.fontSize": 13,
  "terminal.integrated.fontFamily": "'Cascadia Code', Consolas",
  "terminal.integrated.cursorBlinking": true,
  "terminal.integrated.scrollback": 10000,
  "terminal.integrated.tabs.enabled": true,
  "terminal.integrated.splitCwd": "initial",
  
  // ==================== Git 配置 ====================
  
  "git.autofetch": true,
  "git.confirmSync": false,
  "git.enableSmartCommit": true,
  "git.inputValidationLength": 100,
  "git.detectSubmodulesLimit": 10,
  
  // GitLens
  "gitLens.currentLine.enabled": true,
  "gitLens.codeLens.authors.enabled": true,
  "gitLens.codeLens.recentChange.enabled": true,
  
  // ==================== 其他配置 ====================
  
  // 文件图标
  "workbench.iconTheme": "material-icon-theme",
  
  // 颜色主题
  "workbench.colorTheme": "One Dark Pro",
  
  // 标题栏
  "window.title": "${dirty}${activeEditorMedium}${separator}${rootName}",
  
  // 标题栏样式
  "window.titleBarStyle": "native",
  
  // 自动更新
  "update.mode": "none",
  
  // 欢迎页
  "workbench.startupEditor": "none",
  
  // 缩略图
  "breadcrumbs.enabled": true,
  
  // Error Lens
  "errorLens.delay": 0,
  "errorLens.errorMessageEnabled": true,
  "errorLens.warningMessageEnabled": true,
  "errorLens.infoMessageEnabled": true,
  "errorLens.hintMessageEnabled": true,
  
  // TODO Tree
  "todo-tree.general.tags": [
    "TODO",
    "FIXME",
    "BUG",
    "HACK",
    "NOTE",
    "REVIEW",
    "XXX"
  ]
}
```

---

## 总结

本文推荐的 50+ 个扩展涵盖了 ASP.NET 开发的各个方面：

| 分类 | 数量 | 必装程度 |
|------|------|----------|
| C# 开发核心 | 10 | 全部必装 |
| Web 开发 | 8 | 推荐 6 个 |
| Git 集成 | 6 | GitLens 必装 |
| 代码质量 | 8 | 推荐 5 个 |
| 效率提升 | 8 | 按需安装 |
| Docker/K8s | 5 | 容器化项目必装 |
| 数据库 | 5 | 按数据库类型选择 |

**安装建议**：
1. **新手入门**：先安装 C# Dev Kit + GitLens + Material Icon Theme + One Dark Pro + Error Lens
2. **进阶配置**：逐步添加 Web 开发和代码质量扩展
3. **团队共享**：将 settings.json 纳入版本控制，统一团队环境

记住：扩展不是越多越好，根据实际需求选择性安装才是最佳实践。
