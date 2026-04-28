# Cookie 认证实现

> **学习时间**: 约 50 分钟 | **难度**: ⭐⭐⭐ | **前置知识**: 中间件概念、认证授权基础

---

## 📌 本节目标

掌握 Cookie 认证的完整工作原理和实现方式，能够独立搭建基于 Cookie 的用户登录系统，理解 Session Cookie 与 Persistent Cookie 的区别，并做好 CSRF 防护。

---

## 一、Cookie 是什么？先搞清楚基础概念

### 1.1 Cookie 的本质

**Cookie 就是服务器让浏览器保存的一小段文本数据。**

生活类比：想象你去一家咖啡店办会员卡：

```
┌─────────────────────────────────────────────┐
│           Cookie 的工作原理（类比）              │
│                                             │
│   你(浏览器) ──► 咖啡店(服务器)               │
│        │                                    │
│        ▼                                    │
│   第一次光临：注册成为会员                    │
│        │                                    │
│        ▼                                    │
│   咖啡店给你一张会员卡(Cookie)               │
│   卡上写着: "VIP-888, 张三, 2024-12-31到期"    │
│        │                                    │
│        ▼                                    │
│   下次来店：出示会员卡                       │
│   店员看卡 → 确认身份 → 提供会员服务           │
│                                             │
└─────────────────────────────────────────────┘
```

### 1.2 Session Cookie vs Persistent Cookie

| 特性 | Session Cookie（会话 Cookie） | Persistent Cookie（持久化 Cookie） |
|------|------------------------------|----------------------------------|
| **生命周期** | 浏览器关闭即失效 | 设置了明确的过期时间 |
| **存储位置** | 内存中 | 磁盘上 |
| **典型用途** | 保持登录状态（本次会话） | "记住我"功能（下次自动登录） |
| **安全性** | 相对较高（关闭即清除） | 较低（长期存在有被盗风险） |
| **Expires 属性** | 无（或 Session） | 有具体日期时间 |

```csharp
// 配置示例：Session Cookie（默认行为）
options.ExpireTimeSpan = TimeSpan.FromMinutes(30);  // 滑动过期
options.IsPersistent = false;                        // 非持久化

// 配置示例：Persistent Cookie（记住我功能）
options.ExpireTimeSpan = TimeSpan.FromDays(30);      // 30天有效
options.IsPersistent = true;                          // 持久化存储
```

---

## 二、Cookie 认证工作原理（完整流程）

### 2.1 时序图：从登录到访问受保护资源

```mermaid
sequenceDiagram
    participant B as 浏览器(Client)
    participant S as ASP.NET Core Server
    participant DB as 数据库

    Note over B,DB: === 登录流程 ===
    B->>S: POST /Account/Login<br/>{username, password}
    S->>DB: 查询用户并验证密码
    DB-->>S: 返回用户信息
    S->>S: 创建 ClaimsPrincipal
    S->>S: 生成加密的 Auth Cookie
    S-->>B: Set-Cookie: .AspNetCore.Cookies=xxx; HttpOnly; Secure; SameSite=Strict

    Note over B,DB: === 后续请求流程 ===
    B->>S: GET /Dashboard<br/>Cookie: .AspNetCore.Cookies=xxx
    S->>S: AuthenticationMiddleware 解密 Cookie
    S->>S: 还原 ClaimsPrincipal
    S->>S: 填充 HttpContext.User
    S-->>B: 返回 Dashboard 页面（已认证）

    Note over B,DB: === 登出流程 ===
    B->>S: POST /Account/Logout
    S-->>B: Set-Cookie: .AspNetCore.Cookies=; Expires=过去时间
```

### 2.2 Cookie 里存了什么？

Cookie 认证方案中，Cookie 值是一个经过加密和签名的数据包，包含以下信息：

```
┌──────────────────────────────────────────────────┐
│          Cookie 内容结构（加密前）                    │
│                                                  │
│  {                                               │
│    ".AspNetCore.Cookies": {                      │
│      "AuthenticationType": "Cookies",            │
│      "NameClaimType": "http://.../identity/name",│
│      "RoleClaimType": "http://.../identity/role",│
│      "Claims": [                                 │
│        {"Type": "nameid", "Value": "user-123"},  │
│        {"Type": "name", "Value": "张三"},         │
│        {"Type": "role", "Value": "Admin"}        │
│      ],                                          │
│      "IssuedUtc": "2024-01-15T10:00:00Z",         │
│      "ExpiresUtc": "2024-01-15T10:30:00Z"         │
│    }                                             │
│  }                                               │
│                                                  │
│  🔒 整个内容被 DataProtection 加密 + 签名          │
└──────────────────────────────────────────────────┘
```

> **安全要点**：用户无法篡改 Cookie 内容（因为有签名），也无法读取敏感信息（因为加密）。但要注意 Cookie 体积不宜过大，否则会影响每次请求的性能。

---

## 三、AddCookie() 配置详解

### 3.1 最小配置示例

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// 添加 Cookie 认证服务
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        // Cookie 名称（默认值）
        options.Cookie.Name = ".AspNetCore.Cookies";

        // 登录页面路径（未认证时重定向到此）
        options.LoginPath = "/Account/Login";

        // 禁止访问页面路径（无权限时重定向）
        options.AccessDeniedPath = "/Account/AccessDenied";

        // 登出后重定向路径
        options.LogoutPath = "/Account/Logout";

        // 过期时间（滑动窗口）
        options.ExpireTimeSpan = TimeSpan.FromMinutes(30);

        // 是否持久化（"记住我"功能）
        options.IsPersistent = false;

        // 滑动过期（每次活动都延长有效期）
        options.SlidingExpiration = true;
    });

var app = builder.Build();

app.UseAuthentication();    // 必须在 UseAuthorization 之前
app.UseAuthorization();

app.Run();
```

### 3.2 完整配置选项一览

```csharp
.AddCookie(options =>
{
    // ========== Cookie 基本设置 ==========
    options.Cookie.Name = ".AspNetCore.MyApp";       // 自定义 Cookie 名称
    options.Cookie.Path = "/";                        // Cookie 有效路径
    options.Cookie.Domain = "example.com";            // 有效域名（子域名共享）
    options.Cookie.HttpOnly = true;                   // 防止 JavaScript 访问 ✅
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // 仅 HTTPS 传输 ✅
    options.Cookie.SameSite = SameSiteMode.Strict;    // 防 CSRF 攻击 ✅

    // ========== 过期策略 ==========
    options.ExpireTimeSpan = TimeSpan.FromHours(8);   // 绝对过期时间
    options.SlidingExpiration = true;                 // 启用滑动过期
    options.IsPersistent = false;                     // 非"记住我"模式

    // ========== 路径配置 ==========
    options.LoginPath = "/Account/Login";             // 未登录重定向
    options.LogoutPath = "/Account/Logout";           // 登出路径
    options.AccessDeniedPath = "/Account/Forbidden";  // 无权限重定向
    options.ReturnUrlParameter = "returnUrl";         // returnUrl 参数名

    // ========== 安全设置 ==========
    // 拒绝过期的 SecurityStamp（用于检测密码更改等）
    // options.Events.OnValidatePrincipal = ...

    // Cookie 创建时的回调
    options.Events.OnSigningIn = context =>
    {
        Console.WriteLine($"用户 {context.Principal.Identity?.Name} 正在登录");
        return Task.CompletedTask;
    };

    // Cookie 创建完成后的回调
    options.Events.OnSignedIn = context =>
    {
        _logger.LogInformation($"用户登录成功: {context.Principal.Identity?.Name}");
        return Task.CompletedTask;
    };

    // 登出回调
    options.Events.OnSigningOut = context =>
    {
        _logger.LogInformation($"用户登出: {context.Principal.Identity?.Name}");
        return Task.CompletedTask;
    };

    // 验证 Principal 的回调（可用于检查账户是否被禁用等）
    options.Events.OnValidatePrincipal = async context =>
    {
        var userId = context.Principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId == null)
        {
            context.RejectPrincipal();
            await context.HttpContext.SignOutAsync();
            return;
        }

        // 检查用户是否被禁用
        var user = await _userService.GetUserByIdAsync(userId);
        if (user == null || !user.IsActive)
        {
            context.RejectPrincipal();
            await context.HttpContext.SignOutAsync();
        }
    };
});
```

### 3.3 Cookie 安全属性详解

```csharp
options.Cookie.HttpOnly = true;
// ✅ 推荐：防止 JavaScript 通过 document.cookie 读取
// 防御 XSS 攻击窃取 Cookie

options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
// ✅ 推荐：Cookie 只通过 HTTPS 传输
// 防止中间人攻击截获 Cookie

options.Cookie.SameSite = SameSiteMode.Lax;
// ✅ 防止 CSRF 跨站请求伪造攻击
// Strict: 完全禁止跨站携带（最安全，但可能影响某些场景）
// Lax: 允许顶级导航 GET 请求携带（推荐用于一般 Web 应用）
// None: 允许跨站携带（需要配合 Secure 使用，适用于 API 场景）

options.Cookie.IsEssential = true;
// 标记为必要 Cookie（即使用户拒绝非必要 Cookie 也会保留）
// 用于 GDPR 合规
```

---

## 四、完整的登录/登出实现

### 4.1 AccountController 完整代码

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

namespace MyApp.Controllers;

public class AccountController : Controller
{
    private readonly ILogger<AccountController> _logger;

    public AccountController(ILogger<AccountController> logger)
    {
        _logger = logger;
    }

    // ==================== 登录页面 ====================

    [HttpGet]
    [AllowAnonymous]
    public IActionResult Login(string? returnUrl = null)
    {
        // 清除临时的错误消息
        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    [HttpPost]
    [AllowAnonymous]
    [ValidateAntiForgeryToken]  // ⚠️ 必须加！防 CSRF
    public async Task<IActionResult> Login(LoginViewModel model, string? returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;

        if (!ModelState.IsValid)
            return View(model);

        // 1. 验证用户凭据（这里应该是数据库查询+密码验证）
        var user = await ValidateUserAsync(model.Username, model.Password);
        if (user == null)
        {
            ModelState.AddModelError("", "用户名或密码错误");
            _logger.LogWarning("登录失败: 用户名 {Username}", model.Username);
            return View(model);
        }

        // 2. 检查账户状态
        if (!user.IsActive)
        {
            ModelState.AddModelError("", "该账户已被禁用");
            return View(model);
        }

        // 3. 创建用户的 Claims
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim("DisplayName", user.DisplayName),
            new Claim("LastLoginTime", DateTime.UtcNow.ToString("O"))
        };

        // 4. 添加角色 Claims
        foreach (var role in user.Roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        // 5. 创建 ClaimsIdentity
        var claimsIdentity = new ClaimsIdentity(
            claims,
            CookieAuthenticationDefaults.AuthenticationScheme);

        // 6. 创建 AuthenticationProperties（可配置额外选项）
        var authProperties = new AuthenticationProperties
        {
            // "记住我" 功能
            IsPersistent = model.RememberMe,

            // 绝对过期时间（可选，会覆盖 ExpireTimeSpan）
            // ExpiresUtc = DateTimeOffset.UtcNow.AddDays(30),

            // 发起登录请求的 URI（用于登录后跳转回原页面）
            RedirectUri = returnUrl
        };

        // 7. 执行登录（写入 Cookie）
        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            new ClaimsPrincipal(claimsIdentity),
            authProperties);

        _logger.LogInformation("用户 {Username} 登录成功", user.Username);

        // 8. 重定向到目标页面或首页
        if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
        {
            return LocalRedirect(returnUrl);
        }
        return RedirectToAction("Index", "Home");
    }

    // ==================== 登出 ====================

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Logout()
    {
        // 记录登出前的用户信息
        var userName = User.Identity?.Name;
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);

        _logger.LogInformation("用户 {UserName} 已登出", userName);
        return RedirectToAction("Index", "Home");
    }

    // ==================== 拒绝访问页面 ====================

    [HttpGet]
    public IActionResult AccessDenied()
    {
        return View();
    }

    // ==================== 辅助方法 ====================

    private async Task<User?> ValidateUserAsync(string username, string password)
    {
        // TODO: 替换为实际的数据库查询和密码哈希验证
        // 示例代码：
        // var user = await _dbContext.Users
        //     .Include(u => u.UserRoles)
        //     .ThenInclude(ur => ur.Role)
        //     .FirstOrDefaultAsync(u => u.Username == username);
        //
        // if (user == null) return null;
        //
        // var result = _passwordHasher.VerifyHashedPassword(user, user.PasswordHash, password);
        // if (result != PasswordVerificationResult.Success) return null;
        //
        // return user;

        // 模拟返回（仅用于演示）
        if (username == "admin" && password == "Admin@123")
        {
            return new User
            {
                Id = 1,
                Username = "admin",
                Email = "admin@example.com",
                DisplayName = "系统管理员",
                IsActive = true,
                Roles = new List<string> { "Admin", "User" }
            };
        }
        return null;
    }
}

// ViewModel
public class LoginViewModel
{
    [Required(ErrorMessage = "请输入用户名")]
    [Display(Name = "用户名")]
    public string Username { get; set; } = string.Empty;

    [Required(ErrorMessage = "请输入密码")]
    [DataType(DataType.Password)]
    [Display(Name = "密码")]
    public string Password { get; set; } = string.Empty;

    [Display(Name = "记住我")]
    public bool RememberMe { get; set; }
}

// User 模型（简化版）
public class User
{
    public int Id { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string DisplayName { get; set; } = string.Empty;
    public bool IsActive { get; set; }
    public List<string> Roles { get; set; } = new();
}
```

### 4.2 登录视图 (Login.cshtml)

```razor
@model LoginViewModel
@{
    ViewData["Title"] = "登录";
}

<div class="row justify-content-center">
    <div class="col-md-6 col-lg-4">
        <div class="card shadow">
            <div class="card-body p-4">
                <h2 class="text-center mb-4">用户登录</h2>

                <form asp-action="Login" asp-controller="Account" method="post">
                    <!-- AntiForgery Token -->
                    @Html.AntiForgeryToken()

                    <!-- ReturnUrl 隐藏字段 -->
                    <input type="hidden" name="returnUrl" value="@ViewData["ReturnUrl"]" />

                    <!-- 用户名 -->
                    <div class="mb-3">
                        <label asp-for="Username" class="form-label"></label>
                        <input asp-for="Username" class="form-control"
                               placeholder="请输入用户名" autofocus />
                        <span asp-validation-for="Username" class="text-danger"></span>
                    </div>

                    <!-- 密码 -->
                    <div class="mb-3">
                        <label asp-for="Password" class="form-label"></label>
                        <input asp-for="Password" class="form-control"
                               placeholder="请输入密码" />
                        <span asp-validation-for="Password" class="text-danger"></span>
                    </div>

                    <!-- 记住我 -->
                    <div class="mb-3 form-check">
                        <input asp-for="RememberMe" class="form-check-input" />
                        <label asp-for="RememberMe" class="form-check-label">记住我</label>
                    </div>

                    <!-- 错误消息显示 -->
                    @if (!ViewData.ModelState.IsValid && ViewData.ModelState.ContainsKey(""))
                    {
                        <div class="alert alert-danger">
                            @Html.ValidationSummary(false)
                        </div>
                    }

                    <!-- 提交按钮 -->
                    <div class="d-grid">
                        <button type="submit" class="btn btn-primary btn-lg">
                            登 录
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### 4.3 在 Controller 中使用 [Authorize]

```csharp
using Microsoft.AspNetCore.Authorization;

public class DashboardController : Controller
{
    // 所有 Action 都需要认证
    [Authorize]
    public class DashboardController : Controller
    {
        public IActionResult Index()
        {
            // 可以直接访问 User 对象
            var userName = User.Identity?.Name;
            var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

            ViewBag.UserName = userName;
            return View();
        }

        // 仅管理员可访问
        [Authorize(Roles = "Admin")]
        public IActionResult AdminPanel()
        {
            return View();
        }

        // 管理员或编辑者都可访问
        [Authorize(Roles = "Admin,Editor")]
        public IActionResult ContentManagement()
        {
            return View();
        }

        // 公开页面（覆盖类级别的 Authorize）
        [AllowAnonymous]
        public IActionResult PublicInfo()
        {
            return View();
        }
    }
}
```

---

## 五、CSRF 防护（AntiForgery）

### 5.1 什么是 CSRF？

**CSRF（Cross-Site Request Forgery，跨站请求伪造）** 是一种常见的 Web 攻击。

```
正常情况:
你(已登录) ──► 你的网站 ──► 提交表单（带 Cookie）──► 服务器处理 ✓

CSRF 攻击:
恶意网站 ──► 你的浏览器（自动发送你的 Cookie）──► 你的网站 ──► 服务器误以为是你的操作 ✗
```

**生活类比**：有人冒充你的签名在文件上盖章——因为印章（Cookie）一直在你身上带着。

### 5.2 ASP.NET Core 的防 CSRF 机制

ASP.NET Core 默认自动启用 CSRF 保护！

```csharp
// Program.cs - 自动防 CSRF（默认已开启）
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";       // AJAX 请求使用的 Header 名
    options.FormFieldName = "__RequestVerificationToken"; // 表单字段名
    options.Cookie.Name = ".AspNetCore.Antiforgery.xxx";
    options.Cookie.SameSite = SameSiteMode.Strict;
});

var app = builder.Build();
app.UseAntiforgery();  // 启用反伪造中间件
```

### 5.3 在视图中使用

```html
<!-- Razor 视图中的三种写法 -->

<!-- 方式一：HTML Helper（推荐） -->
<form method="post">
    @Html.AntiForgeryToken()
    <!-- 表单内容 -->
</form>

<!-- 方式二：Tag Helper（自动添加，推荐） -->
<form asp-action="Login" method="post">
    <!-- Tag Helper 会自动注入 __RequestVerificationToken -->
    <!-- 表单内容 -->
</form>

<!-- 方式三：手动获取 Token -->
<input type="hidden" name="__RequestVerificationToken"
       value="@this.Context.RequestAntiforgeryTokens().FormToken" />
```

### 5.4 在 API/AJAX 中使用

```javascript
// 前端 JavaScript：从 Cookie/Meta 标签获取 Token 并附加到请求头
function getCsrfToken() {
    // 从 meta 标签获取（推荐方式）
    const metaTag = document.querySelector('meta[name="csrf-token"]');
    return metaTag ? metaTag.content : '';
}

// fetch 示例
fetch('/api/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-TOKEN': getCsrfToken()  // 在 Header 中传递
    },
    body: JSON.stringify(data)
});
```

```html
<!-- 在 Layout 或页面 head 中放置 Token -->
<head>
    <meta name="csrf-token"
          content="@this.Context.RequestAntiforgeryTokens().RequestToken" />
</head>
```

---

## 六、Session 存储方式选择

当应用需要在服务端存储会话数据时，有以下几种选择：

### 6.1 内存存储（开发环境用）

```csharp
// Program.cs
builder.Services.AddDistributedMemoryCache();  // 内存缓存
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

app.UseSession();
```

**优点**：零配置，开箱即用。
**缺点**：重启丢失，不支持多实例部署，不适合生产环境。

### 6.2 分布式缓存（Redis - 生产推荐）

```csharp
// 安装包: Microsoft.Extensions.Caching.StackExchangeRedis
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MyApp_Session_";
});

builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
    options.Cookie.HttpOnly = true;
    options.Cookie.SameSite = SameSiteMode.Lax;
});
```

**优点**：支持多服务器部署，性能优秀，支持数据持久化。
**缺点**：需要 Redis 基础设施。

### 6.3 SQL Server 数据库

```csharp
// 安装包: Microsoft.Extensions.Caching.SqlServer
// 先执行工具创建表:
// dotnet sql-cache create "连接字符串" dbo SessionCache

builder.Services.AddSqlServerCache(options =>
{
    options.ConnectionString = builder.Configuration.GetConnectionString("DefaultConnection");
    options.SchemaName = "dbo";
    options.TableName = "SessionCache";
    options.ExpiredItemsDeletionInterval = TimeSpan.FromMinutes(30);
});
```

**优点**：无需额外基础设施，利用现有数据库。
**缺点**：性能不如 Redis，频繁读写可能造成数据库压力。

### 6.4 选择建议

```
┌────────────────────────────────────────────────────┐
│              Session 存储选择决策树                  │
│                                                    │
│   开发/测试环境？                                   │
│      ├── 是 → DistributedMemoryCache（最简单）       │
│      └── 否 ↓                                      │
│                                                    │
│   单服务器部署？                                     │
│      ├── 是 → In-Memory 或 SQL Server              │
│      └── 否（多服务器/集群）↓                        │
│                                                    │
│   已有 Redis 基础设施？                              │
│      ├── 是 → StackExchangeRedisCache（最佳选择）✅  │
│      └── 否 → SqlServerCache 或部署 Redis           │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 七、Cookie 认证优缺点分析

### 7.1 优点

| 优势 | 说明 |
|------|------|
| **实现简单** | ASP.NET Core 内置支持，几行代码即可完成 |
| **用户体验好** | 自动携带 Cookie，无需前端手动管理 Token |
| **兼容性强** | 所有浏览器都完美支持 |
| **适合传统 MVC** | 服务端渲染的应用天然适配 |

### 7.2 缺点与应对

| 缺点 | 影响 | 应对措施 |
|------|------|----------|
| **CSRF 风险** | 需要额外的防护机制 | 使用 `[ValidateAntiForgeryToken]` 和 `UseAntiforgery()` |
| **移动端不友好** | 原生 App 处理 Cookie 较复杂 | 移动端优先考虑 JWT Token |
| **跨域问题** | Cookie 受同源策略限制 | 配置 CORS + SameSite=None + Secure |
| **体积限制** | 每个 Cookie 约 4KB 限制 | 不要在 Cookie 中存太多 Claims |
| **难以撤销** | 一旦签发，只能等过期 | 结合服务端黑名单机制 |

---

## 八、完整示例：Cookie 认证 Web 应用

### 8.1 项目结构

```
CookieAuthDemo/
├── Program.cs                  # 配置服务和中间件
├── appsettings.json
├── Controllers/
│   ├── HomeController.cs       # 公开页面
│   ├── AccountController.cs    # 登录/登出
│   └── DashboardController.cs  # 受保护页面
├── Views/
│   ├── Home/
│   │   └── Index.cshtml
│   ├── Account/
│   │   ├── Login.cshtml
│   │   └── AccessDenied.cshtml
│   └── Dashboard/
│       └── Index.cshtml
└── Models/
    └── LoginViewModel.cs
```

### 8.2 Program.cs 完整配置

```csharp
using Microsoft.AspNetCore.Authentication.Cookies;
using System.Security.Claims;

var builder = WebApplication.CreateBuilder(args);

// 添加 MVC/Razor 服务
builder.Services.AddControllersWithViews();

// 添加 Cookie 认证
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.Name = ".AspNetCore.DemoAuth";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = builder.Environment.IsDevelopment()
            ? CookieSecurePolicy.SameAsRequest
            : CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;

        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";

        options.ExpireTimeSpan = TimeSpan.FromHours(2);
        options.SlidingExpiration = true;

        // 登录成功日志
        options.Events.OnSignedIn = ctx =>
        {
            Console.WriteLine($"[登录] 用户: {ctx.Principal.Identity?.Name}");
            return Task.CompletedTask;
        };

        // 登出日志
        options.Events.OnSigningOut = ctx =>
        {
            Console.WriteLine($"[登出] 用户: {ctx.Principal.Identity?.Name}");
            return Task.CompletedTask;
        };
    });

// 添加防 CSRF
builder.Services.AddAntiforgery();

var app = builder.Build();

// HTTP 严格传输安全（生产环境必须）
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAntiforgery();

// ⚠️ 顺序关键！
app.UseAuthentication();    // 先认证
app.UseAuthorization();      // 后授权

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

---

## 九、安全最佳实践清单

### DO ✅

- **DO** 始终设置 `HttpOnly=true` 防止 JavaScript 读取 Cookie
- **DO** 生产环境务必设置 `SecurePolicy=Always` 强制 HTTPS
- **DO** 合理配置 `SameSite` 属性（Lax 或 Strict）
- **DO** 所有 POST/PUT/DELETE 操作都要加 `[ValidateAntiForgeryToken]`
- **DO** 使用 `DataProtection` 加密 Cookie 内容（框架默认已做）
- **DO** 设置合理的过期时间和 SlidingExpiration
- **DO** 在 OnValidatePrincipal 回调中检查账户是否被禁用/删除
- **DO** 登出时彻底清理 Cookie 和 Session

### DON'T ❌

- **DON'T** 不要在 Cookie 中存储敏感信息（密码、银行卡号等）
- **DON'T** 不要忽略 CSRF 防护——即使是 API 也要处理
- **DON'T** 不要在 URL 中传递 Session ID 或认证信息
- **DON'T** 不要使用过长的 Cookie 过期时间（"记住我"也不应超过 30 天）
- **DON'T** 不要信任客户端传来的 Role 或 Permission 信息
- **DON'T** 不要忘记在登出时调用 `SignOutAsync()`（不能只删 Cookie）
- **DON'T** 不要在多个子域间随意共享认证 Cookie（除非明确需要且做好安全评估）
- **DON'T** 不要把 Cookie 用于纯 API 项目（应使用 JWT Bearer Token）

---

## 十、练习题

### 练习 1：配置分析
分析以下 Cookie 配置的安全性，指出至少 3 个可以改进的地方：
```csharp
options.Cookie.HttpOnly = false;
options.Cookie.SecurePolicy = CookieSecurePolicy.Never;
options.ExpireTimeSpan = TimeSpan.FromDays(365);
options.SlidingExpiration = false;
options.Cookie.SameSite = SameSiteMode.None;
```

### 练习 2：登录流程完善
在上述登录代码基础上，增加以下功能：
- 登录失败 5 次后锁定账户 15 分钟
- 记录每次登录的 IP 地址和时间
- 密码错误时不提示"用户名或密码错误"，而是分别提示

### 练习 3："记住我"实现
实现一个完善的"记住我"功能：
- 勾选后 Cookie 有效期 30 天
- 不勾选则关闭浏览器即失效
- 在受保护页面显示当前登录状态和剩余有效时间

### 练习 4：CSRF 防护实践
创建一个简单的 AJAX 表单提交，确保正确传递 CSRF Token。如果遗漏了 Token，观察会发生什么错误？

### 练习 5：Session 选型
为一个即将上线的高并发电商网站选择合适的 Session 存储方案，说明理由，并给出配置代码。

---

## 十一、延伸阅读

- [Microsoft Docs: ASP.NET Core 中的 Cookie 身份验证](https://docs.microsoft.com/zh-cn/aspnet/core/security/authentication/cookie)
- [Microsoft Docs: 防止跨站点请求伪造 (XSRF/CSRF) 攻击](https://docs.microsoft.com/zh-cn/aspnet/core/security/anti-request-forgery)
- [OWASP: Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [SameSite Cookies Explained](https://web.dev/samesite-cookies-explained/)
- [ASP.NET Core Data Protection 概述](https://docs.microsoft.com/zh-cn/aspnet/core/security/data-protection/introduction)

---

> **下一节预告**：我们将学习 **JWT Bearer Token 详解**，了解这种无状态的认证方式如何解决跨域和移动端认证问题，以及它与 Cookie 认证的优劣对比。
