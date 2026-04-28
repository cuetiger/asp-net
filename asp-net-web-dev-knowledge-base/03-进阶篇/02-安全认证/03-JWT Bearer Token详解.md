# JWT Bearer Token 详解

> **学习时间**: 约 55 分钟 | **难度**: ⭐⭐⭐ | **前置知识**: 认证授权基础、HTTP 协议基础、JSON 格式

---

## 📌 本节目标

深入理解 JSON Web Token (JWT) 的结构和工作原理，掌握在 ASP.NET Core 中配置和使用 JWT Bearer 认证，能够独立实现基于 Token 的前后端分离认证系统。

---

## 一、JSON Web Token 结构

### 1.1 JWT 是什么？

**JWT（JSON Web Token）** 是一种开放标准（RFC 7519），用于在各方之间以 JSON 对象的形式安全地传输信息。它是目前最流行的无状态认证方案。

生活类比：JWT 就像一张**加密的入场券**

```
┌─────────────────────────────────────────────────────┐
│              JWT = 入场券（类比）                      │
│                                                     │
│   Header（票面信息）                                  │
│   ├── 票种: VIP                                    │
│   └── 编码方式: 二维码                               │
│                                                     │
│   Payload（持有人信息）                                │
│   ├── 持有人: 张三                                   │
│   ├── 座位号: A-12-05                               │
│   └── 有效期: 2024-12-31                            │
│                                                     │
│   Signature（防伪印章）                                │
│   └── HMAC-SHA256(Header + Payload + 私钥)           │
│                                                     │
│   最终格式: xxxxx.yyyyy.zzzzz                        │
│            (Base64URL编码的三段，用点分隔)               │
└─────────────────────────────────────────────────────┘
```

### 1.2 JWT 的三部分结构

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IuW8oOS4iSIsImFkbWluIjp0cnVlfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
├─────────────────┼──────────────────────────────────┼──────────────────────────┤
      Header              Payload                          Signature
     (头部)               (载荷)                           (签名)
```

#### 第一部分：Header（头部）

```json
{
  "alg": "HS256",        // 签名算法：HMAC SHA256
  "typ": "JWT"          // Token 类型：JWT
}
```

```csharp
// 对应的 C# 表示
var header = new { alg = "HS256", typ = "JWT" };
var headerBase64 = Base64UrlEncode(JsonSerializer.Serialize(header));
// 结果: eyJhbGciOiJIUzI1NiJ9
```

#### 第二部分：Payload（载荷）

```json
{
  "sub": "1234567890",    // Subject: 用户唯一标识
  "name": "张三",          // 用户名
  "admin": true,          // 是否管理员
  "iat": 1705312800,      // Issued At: 签发时间戳
  "exp": 1705316400       // Expiration: 过期时间戳
}
```

> **注意**：Payload 只是 Base64 编码，不是加密！任何人都可以解码读取。所以绝对不要在 Payload 中放密码、银行卡号等敏感信息。

#### 第三部分：Signature（签名）

```
Signature = HMACSHA256(
    Base64Url(Header) + "." + Base64Url(Payload),
    Your-256-Bit-Secret-Key
)
```

签名的目的：确保 Token 在传输过程中没有被篡改。

### 1.3 解码一个真实的 JWT

```csharp
using System.Text;
using System.Text.Json;

public class JwtDecoder
{
    /// <summary>
    /// 解码 JWT（仅解码，不验证签名）
    /// </summary>
    public static JwtPayload Decode(string token)
    {
        var parts = token.Split('.');
        if (parts.Length != 3)
            throw new ArgumentException("Invalid JWT format");

        // Base64Url 解码
        var headerJson = Base64UrlDecode(parts[0]);
        var payloadJson = Base64UrlDecode(parts[1]);

        return new JwtPayload
        {
            Header = JsonSerializer.Deserialize<Dictionary<string, object>>(headerJson)!,
            Claims = JsonSerializer.Deserialize<Dictionary<string, object>>(payloadJson)!,
            Signature = parts[2]
        };
    }

    private static string Base64UrlDecode(string input)
    {
        input = input.Replace('-', '+').Replace('_', '/');
        switch (input.Length % 4)
        {
            case 2: input += "=="; break;
            case 3: input += "="; break;
        }
        var bytes = Convert.FromBase64String(input);
        return Encoding.UTF8.GetString(bytes);
    }
}

public class JwtPayload
{
    public Dictionary<string, object> Header { get; set; } = new();
    public Dictionary<string, object> Claims { get; set; } = new();
    public string Signature { get; set; } = string.Empty;

    public override string ToString()
    {
        return $"Header: {JsonSerializer.Serialize(Header)}\n" +
               $"Claims: {JsonSerializer.Serialize(Claims)}\n" +
               $"Signature: {Signature.Substring(0, Math.Min(20, Signature.Length))}...";
    }
}

// 使用示例：
// var decoded = JwtDecoder.Decode(token);
// Console.WriteLine(decoded);
```

---

## 二、JWT vs Session 对比

### 2.1 核心差异对比表

| 特性 | JWT (Token) | Session/Cookie |
|------|-------------|----------------|
| **存储位置** | 客户端（浏览器/本地存储） | 服务端（内存/Redis/数据库） |
| **状态** | 无状态（Stateless） | 有状态（Stateful） |
| **跨域能力** | 天然支持（放在 Authorization 头） | 受同源策略限制 |
| **移动端友好度** | 极好（原生 App 易于处理） | 较差（需要 Cookie 管理） |
| **扩展性** | 极佳（服务端不存状态） | 需要共享 Session 存储 |
| **即时失效** | 困难（需配合黑名单） | 简单（直接删除 Session） |
| **Token 大小** | 较大（几百字节到几KB） | 仅一个 Session ID（几十字节） |
| **安全性** | 取决于密钥管理和传输安全 | 相对成熟稳定 |

### 2.2 架构对比图

```
Session 方式:
┌─────────┐         ┌─────────┐         ┌─────────┐
│  Client │ ──Cookie──► │ Server  │ ──查询──► │ Redis/DB │
│         │ ◄─HTML──── │         │ ◄─数据── │          │
└─────────┘         └─────────┘         └─────────┘
                     ↑
                每次请求都要查 Session

JWT 方式:
┌─────────┐                              ┌─────────┐
│  Client │ ──Authorization: Bearer xxx──►│ Server  │
│         │  (Token 自包含所有信息)          │         │
│         │ ◄─────────Response────────────│         │
└─────────┘                              └─────────┘
                     ↑
             服务端无需查数据库，自验证即可
```

### 3.3 选择建议

```
选择 JWT 的场景:
✅ 前后端分离的 SPA 应用（React/Vue/Angular）
✅ 移动端 App（iOS/Android）
✅ 微服务架构（多个服务共享认证）
✅ 需要跨域访问的 API
✅ 高并发场景（减少服务器压力）

选择 Session/Cookie 的场景:
✅ 传统 MVC/Razor Pages 应用
✅ 需要即时让 Token 失效的场景
✅ 对安全性要求极高的金融系统
✅ 不想处理 Token 管理复杂度的简单应用
```

---

## 三、生成和验证 JWT Token

### 3.1 手动生成 JWT（理解原理）

```csharp
using System.IdentityModel.Tokens.Jwt;
using Microsoft.IdentityModel.Tokens;
using System.Security.Claims;
using System.Security.Cryptography;
using System.Text;

public class JwtTokenService
{
    private readonly string _secretKey;
    private readonly string _issuer;
    private readonly string _audience;
    private readonly int _expirationMinutes;

    public JwtTokenService(IConfiguration config)
    {
        _secretKey = config["Jwt:SecretKey"]
            ?? throw new ArgumentNullException("Jwt:SecretKey is required");
        _issuer = config["Jwt:Issuer"] ?? "MyApp";
        _audience = config["Jwt:Audience"] ?? "MyAppClient";
        _expirationMinutes = int.Parse(config["Jwt:ExpirationMinutes"] ?? "60");
    }

    /// <summary>
    /// 生成 JWT Token
    /// </summary>
    public string GenerateToken(UserInfo user)
    {
        // 1. 定义 Claims（声明）
        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id),                    // 用户ID
            new Claim(JwtRegisteredClaimNames.Name, user.Username),             // 用户名
            new Claim(JwtRegisteredClaimNames.Email, user.Email),               // 邮箱
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),  // Token 唯一ID
            new Claim(ClaimTypes.Role, string.Join(",", user.Roles)),           // 角色
            new Claim("Department", user.Department),                           // 自定义声明
            new Claim("Avatar", user.AvatarUrl)                                 // 自定义声明
        };

        // 2. 准备签名密钥
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_secretKey));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        // 3. 创建 Token 描述
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Issuer = _issuer,                   // 签发者
            Audience = _audience,               // 接收者
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(_expirationMinutes),  // 过期时间
            IssuedAt = DateTime.UtcNow,         // 签发时间
            SigningCredentials = credentials    // 签名凭据
        };

        // 4. 生成 Token
        var tokenHandler = new JwtSecurityTokenHandler();
        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }

    /// <summary>
    /// 验证并解析 JWT Token（手动验证，用于非中间件场景）
    /// </summary>
    public ClaimsPrincipal? ValidateToken(string token)
    {
        try
        {
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.UTF8.GetBytes(_secretKey);

            var validationParameters = new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = _issuer,
                ValidateAudience = true,
                ValidAudience = _audience,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero  // 时钟偏差容忍度（默认5分钟）
            };

            var principal = tokenHandler.ValidateToken(
                token, validationParameters, out SecurityToken validatedToken);

            return principal;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Token 验证失败: {ex.Message}");
            return null;
        }
    }

    /// <summary>
    /// 从 Token 中提取过期时间
    /// </summary>
    public DateTime? GetExpiration(string token)
    {
        try
        {
            var handler = new JwtSecurityTokenHandler();
            var jwtToken = handler.ReadJwtToken(token);
            return jwtToken.ValidTo;
        }
        catch
        {
            return null;
        }
    }
}

// 用户信息模型
public class UserInfo
{
    public string Id { get; set; } = string.Empty;
    public string Username { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public List<string> Roles { get; set; } = new();
    public string Department { get; set; } = string.Empty;
    public string AvatarUrl { get; set; } = string.Empty;
}
```

### 3.2 生成 Token 的步骤详解

```mermaid
flowchart TD
    A[用户登录请求] --> B[验证用户名和密码]
    B --> C{验证通过?}
    C -->|否| D[返回错误信息]
    C -->|是| E[构建 Claims 列表]

    E --> F[准备签名密钥<br/>SymmetricSecurityKey]
    F --> G[创建 SigningCredentials<br/>指定算法 HS256]
    G --> H[创建 SecurityTokenDescriptor<br/>设置 Issuer/Audience/Expires 等]
    H --> I[JwtSecurityTokenHandler.CreateToken]
    I --> J[WriteToken 序列化为字符串]
    J --> K[返回 JWT 给客户端]

    style A fill:#e1f5fe
    style K fill:#c8e6c9
    style D fill:#ffcdd2
```

### 3.3 验证 Token 的过程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Server as API 服务器

    Client->>Server: GET /api/data<br/>Authorization: Bearer eyJhbG...

    Note over Server: 步骤1: 提取 Token
    Server->>Server: 从 Authorization 头提取 Bearer Token

    Note over Server: 步骤2: 分割三部分
    Server->>Server: Split('.') → [Header, Payload, Signature]

    Note over Server: 步骤3: 验证签名
    Server->>Server: 重新计算 Signature 并比对<br/>确保未被篡改

    Note over Server: 步骤4: 验证 Issuer
    Server->>Server: 检查 iss 字段是否匹配配置值

    Note over Server: 步骤5: 验证 Audience
    Server->>Server: 检查 aud 字段是否匹配配置值

    Note over Server: 步骤6: 验证有效期
    Server->>Server: 检查 exp > 当前时间<br/>检查 iat <= 当前时间

    Note over Server: 步骤7: 构建 ClaimsPrincipal
    Server->>Server: 从 Payload 还原用户身份信息

    alt 全部验证通过
        Server-->>Client: 200 OK + 数据
    else 验证失败
        Server-->>Client: 401 Unauthorized
    end
```

---

## 四、在 ASP.NET Core 中配置 JWT Bearer 认证

### 4.1 appsettings.json 配置

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "Jwt": {
    "SecretKey": "YourSuperSecretKeyThatIsAtLeast32CharactersLong!",
    "Issuer": "MySecureApi",
    "Audience": "MyApiClient",
    "ExpirationMinutes": 60
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=JwtDemoDb"
  }
}
```

> **重要提示**：生产环境的 SecretKey 应该从环境变量或 Azure Key Vault 等安全存储中获取，不要硬编码在配置文件中！

### 4.2 Program.cs 完整配置

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

// ==================== 添加服务 ====================

builder.Services.AddControllers();

// 配置 Swagger 支持 JWT
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "JWT Auth API",
        Version = "v1"
    });

    // 添加 JWT 认证到 Swagger UI
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header. Example: \"Bearer {token}\"",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});

// ==================== 核心：JWT Bearer 认证配置 ====================

var jwtConfig = builder.Configuration.GetSection("Jwt");
var secretKey = jwtConfig["SecretKey"];

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    // ========== 密钥配置 ==========
    options.TokenValidationParameters = new TokenValidationParameters
    {
        // 签名密钥验证
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(secretKey!)),

        // 签发者验证
        ValidateIssuer = true,
        ValidIssuer = jwtConfig["Issuer"],

        // 接收者验证
        ValidateAudience = true,
        ValidAudience = jwtConfig["Audience"],

        // 过期时间验证
        ValidateLifetime = true,

        // 时钟偏差容忍度（考虑服务器间的时间差）
        ClockSkew = TimeSpan.Zero  // 生产环境可设为 0-5 分钟
    };

    // ========== 事件回调 ==========

    // 认证失败时的处理
    options.Events = new JwtBearerEvents
    {
        OnTokenValidated = context =>
        {
            // Token 验证成功后的额外逻辑
            var userId = context.Principal?.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
            Console.WriteLine($"[JWT] Token 验证成功, UserId: {userId}");
            return Task.CompletedTask;
        },

        OnAuthenticationFailed = context =>
        {
            Console.WriteLine($"[JWT] 认证失败: {context.Exception.Message}");

            // 自定义错误响应
            if (context.Exception is SecurityTokenExpiredException)
            {
                context.Response.Headers.Append("Token-Expired", "true");
            }

            return Task.CompletedTask;
        },

        OnChallenge = context =>
        {
            // 跳过默认的处理逻辑
            context.HandleResponse();

            // 自定义 401 响应格式
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            context.Response.ContentType = "application/json";

            var result = JsonSerializer.Serialize(new
            {
                code = 401,
                message = "未授权，请提供有效的 Token",
                timestamp = DateTime.UtcNow
            });

            return context.Response.WriteAsync(result);
        },

        OnForbidden = context =>
        {
            // 自定义 403 响应格式
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            context.Response.ContentType = "application/json";

            var result = JsonSerializer.Serialize(new
            {
                code = 403,
                message = "您没有权限执行此操作",
                timestamp = DateTime.UtcNow
            });

            return context.Response.WriteAsync(result);
        }
    };
});

// 授权服务
builder.Services.AddAuthorization();

// 注册自定义服务
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<JwtTokenService>();

// 数据库（如果使用 EF Core）
// builder.Services.AddDbContext<AppDbContext>(options =>
//     options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// ==================== 配置 HTTP 管道 ====================

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "JWT Auth API v1");
    });
}

app.UseHttpsRedirection();

// ⚠️ 顺序很重要！
app.UseAuthentication();    // 先认证
app.UseAuthorization();      // 后授权

app.MapControllers();

app.Run();
```

### 4.3 Token 参数详解

```csharp
// 这些参数在生成 Token 时设置
new SecurityTokenDescriptor
{
    // --- 签发相关 ---
    Issuer = "MySecureApi",           // 谁签发的这个 Token？
    Audience = "MyApiClient",         // 这个 Token 是给谁用的？

    // --- 时间相关 ---
    IssuedAt = DateTime.UtcNow,       // 什么时候签发的？（iat）
    Expires = DateTime.UtcNow.AddHours(24),  // 什么时候过期？（exp）
    NotBefore = DateTime.UtcNow,      // 什么时候之前不能用？（nbf）

    // --- 身份相关 ---
    Subject = new ClaimsIdentity(claims),  // 用户的 Claims 信息

    // --- 安全相关 ---
    SigningCredentials = credentials       // 签名算法和密钥
};
```

| 参数 | 全称 | 说明 | 示例 |
|------|------|------|------|
| `iss` | Issuer | Token 签发者 | `"MySecureApi"` |
| `aud` | Audience | 目标接收者 | `"MyApiClient"` |
| `sub` | Subject | 用户唯一标识 | `"user-12345"` |
| `jti` | JWT ID | Token 唯一标识（防重放攻击） | `"guid-xxx"` |
| `iat` | Issued At | 签发时间（Unix时间戳） | `1705312800` |
| `exp` | Expiration | 过期时间（Unix时间戳） | `1705399200` |
| `nbf` | Not Before | 生效时间之前不可用 | `1705312800` |

---

## 五、完整的 JWT 登录/鉴权 API 示例

### 5.1 认证控制器 (AuthController.cs)

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;

namespace MyApp.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IAuthService _authService;
    private readonly JwtTokenService _tokenService;
    private readonly ILogger<AuthController> _logger;

    public AuthController(
        IAuthService authService,
        JwtTokenService tokenService,
        ILogger<AuthController> logger)
    {
        _authService = authService;
        _tokenService = tokenService;
        _logger = logger;
    }

    /// <summary>
    /// 用户登录 - 返回 JWT Token
    /// </summary>
    /// <param name="model">登录模型</param>
    /// <returns>Token 和用户基本信息</returns>
    [HttpPost("login")]
    [AllowAnonymous]
    public async Task<ActionResult<LoginResponse>> Login([FromBody] LoginRequest model)
    {
        #region 1. 参数验证
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }
        #endregion

        #region 2. 验证用户凭据
        var user = await _authService.ValidateUserAsync(model.Username, model.Password);
        if (user == null)
        {
            _logger.LogWarning("登录失败: 用户名 {Username}", model.Username);
            return Unauthorized(new ApiResponse(401, "用户名或密码错误"));
        }
        #endregion

        #region 3. 检查账户状态
        if (!user.IsActive)
        {
            return Forbid(new ApiResponse(403, "该账户已被禁用"));
        }

        if (user.IsLockedOut && user.LockoutEnd.HasValue && user.LockoutEnd.Value > DateTime.UtcNow)
        {
            return Forbid(new ApiResponse(403,
                $"账户已锁定，请在 {(user.LockoutEnd.Value - DateTime.UtcNow).TotalMinutes:F0} 分钟后重试"));
        }
        #endregion

        #region 4. 生成 JWT Token
        var userInfo = new UserInfo
        {
            Id = user.Id.ToString(),
            Username = user.Username,
            Email = user.Email,
            Roles = user.Roles.Select(r => r.Name).ToList(),
            Department = user.Department,
            AvatarUrl = user.AvatarUrl
        };

        var token = _tokenService.GenerateToken(userInfo);
        var expiration = _tokenService.GetExpiration(token);
        #endregion

        #region 5. 记录日志 & 返回结果
        _logger.LogInformation("用户 {Username} 登录成功", user.Username);

        // 可选：记录登录历史
        await _authService.RecordLoginHistoryAsync(user.Id, HttpContext.Connection.RemoteIpAddress?.ToString());

        return Ok(new LoginResponse
        {
            Token = token,
            TokenType = "Bearer",
            ExpiresIn = (int)(expiration?.Subtract(DateTime.UtcNow).TotalSeconds ?? 3600),
            User = new UserInfoDto
            {
                Id = user.Id,
                Username = user.Username,
                DisplayName = user.DisplayName,
                Email = user.Email,
                AvatarUrl = user.AvatarUrl,
                Roles = userInfo.Roles
            }
        });
        #endregion
    }

    /// <summary>
    /// 刷新 Token（可选功能）
    /// </summary>
    [HttpPost("refresh")]
    [AllowAnonymous]
    public async Task<ActionResult<LoginResponse>> RefreshToken([FromBody] RefreshTokenRequest model)
    {
        // TODO: 实现 Refresh Token 机制
        // 1. 验证 Refresh Token 是否有效且未过期
        // 2. 检查 Refresh Token 是否已被撤销
        // 3. 生成新的 Access Token 和新的 Refresh Token
        // 4. 使旧的 Refresh Token 失效

        return Ok(new ApiResponse(200, "刷新 Token 功能待实现"));
    }

    /// <summary>
    /// 获取当前用户信息
    /// </summary>
    [HttpGet("me")]
    [Authorize]
    public async Task<ActionResult<UserInfoDto>> GetCurrentUser()
    {
        var userId = User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
        if (userId == null) return Unauthorized();

        var user = await _authService.GetUserByIdAsync(Guid.Parse(userId));
        if (user == null) return NotFound(new ApiResponse(404, "用户不存在"));

        return Ok(new UserInfoDto
        {
            Id = user.Id,
            Username = user.Username,
            DisplayName = user.DisplayName,
            Email = user.Email,
            AvatarUrl = user.AvatarUrl,
            Roles = user.Roles.Select(r => r.Name).ToList()
        });
    }

    /// <summary>
    /// 修改密码
    /// </summary>
    [HttpPost("change-password")]
    [Authorize]
    public async Task<ActionResult> ChangePassword([FromBody] ChangePasswordRequest model)
    {
        var userId = User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;
        if (userId == null) return Unauthorized();

        var result = await _authService.ChangePasswordAsync(
            Guid.Parse(userId),
            model.OldPassword,
            model.NewPassword);

        if (!result.Succeeded)
        {
            return BadRequest(new ApiResponse(400, result.Errors.FirstOrDefault()?.Description ?? "修改失败"));
        }

        _logger.LogInformation("用户 {UserId} 修改了密码", userId);
        return Ok(new ApiResponse(200, "密码修改成功"));
    }

    /// <summary>
    /// 登出（客户端删除 Token 即可，服务端可选黑名单机制）
    /// </summary>
    [HttpPost("logout")]
    [Authorize]
    public ActionResult Logout()
    {
        // JWT 无状态，服务端无法主动使 Token 失效
        // 方案1：客户端删除 Token（推荐，最简单）
        // 方案2：将 Token 加入黑名单（Redis 缓存，有过期时间）
        // 方案3：缩短 Token 有效期 + 使用 Refresh Token

        return Ok(new ApiResponse(200, "登出成功，请客户端删除 Token"));
    }
}
```

### 5.2 受保护的资源控制器示例

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]  // 所有接口都需要认证
public class ArticlesController : ControllerBase
{
    private readonly IArticleService _articleService;
    private readonly ILogger<ArticlesController> _logger;

    public ArticlesController(IArticleService articleService, ILogger<ArticlesController> logger)
    {
        _articleService = articleService;
        _logger = logger;
    }

    // 所有已认证用户都可以获取文章列表
    [HttpGet]
    public async Task<ActionResult<PagedResult<ArticleDto>>> GetArticles(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        var userId = User.FindFirstValue(JwtRegisteredClaimNames.Sub);
        _logger.LogInformation("用户 {UserId} 获取文章列表", userId);

        var result = await _articleService.GetPagedAsync(page, pageSize);
        return Ok(result);
    }

    // 只有管理员可以创建文章
    [HttpPost]
    [Authorize(Roles = "Admin,Editor")]
    public async Task<ActionResult<ArticleDto>> CreateArticle([FromBody] CreateArticleRequest request)
    {
        var authorId = User.FindFirstValue(JwtRegisteredClaimNames.Sub);
        var authorName = User.FindFirstValue(JwtRegisteredClaimNames.Name);

        var article = await _articleService.CreateAsync(new ArticleCreateModel
        {
            Title = request.Title,
            Content = request.Content,
            AuthorId = Guid.Parse(authorId!),
            AuthorName = authorName!
        });

        return CreatedAtAction(nameof(GetArticle), new { id = article.Id }, article);
    }

    // 只能编辑自己的文章（或管理员）
    [HttpPut("{id}")]
    public async Task<ActionResult> UpdateArticle(int id, [FromBody] UpdateArticleRequest request)
    {
        var article = await _articleService.GetByIdAsync(id);
        if (article == null) return NotFound();

        var currentUserId = User.FindFirstValue(JwtRegisteredClaimNames.Sub);
        var isAdmin = User.IsInRole("Admin");

        // 权限检查：作者本人 或 管理员
        if (article.AuthorId.ToString() != currentUserId && !isAdmin)
        {
            return Forbid(new ApiResponse(403, "只能编辑自己创建的文章"));
        }

        await _articleService.UpdateAsync(id, request.Title, request.Content);
        return NoContent();
    }

    // 仅管理员可以删除文章
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult> DeleteArticle(int id)
    {
        var exists = await _articleService.ExistsAsync(id);
        if (!exists) return NotFound();

        await _articleService.DeleteAsync(id);
        _logger.LogInformation("管理员 {AdminId} 删除了文章 {ArticleId}",
            User.FindFirstValue(JwtRegisteredClaimNames.Sub), id);

        return NoContent();
    }
}
```

### 5.3 DTO 和响应模型

```csharp
// ====== 请求模型 ======

public class LoginRequest
{
    [Required(ErrorMessage = "请输入用户名")]
    public string Username { get; set; } = string.Empty;

    [Required(ErrorMessage = "请输入密码")]
    [MinLength(6, ErrorMessage = "密码至少6个字符")]
    public string Password { get; set; } = string.Empty;
}

public class RefreshTokenRequest
{
    public string RefreshToken { get; set; } = string.Empty;
}

public class ChangePasswordRequest
{
    public string OldPassword { get; set; } = string.Empty;
    public string NewPassword { get; set; } = string.Empty;
    public string ConfirmPassword { get; set; } = string.Empty;
}

// ====== 响应模型 ======

public class ApiResponse
{
    public int Code { get; set; }
    public string Message { get; set; }
    public object? Data { get; set; }

    public ApiResponse(int code, string message, object? data = null)
    {
        Code = code;
        Message = message;
        Data = data;
    }
}

public class LoginResponse
{
    public string Token { get; set; } = string.Empty;
    public string TokenType { get; set; } = "Bearer";
    public int ExpiresIn { get; set; }  // 过期秒数
    public UserInfoDto? User { get; set; }
}

public class UserInfoDto
{
    public Guid Id { get; set; }
    public string Username { get; set; } = string.Empty;
    public string DisplayName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? AvatarUrl { get; set; }
    public List<string> Roles { get; set; } = new();
}
```

---

## 六、Token 刷新机制（Refresh Token）

### 6.1 为什么需要 Refresh Token？

```
问题：JWT 一旦签发就无法撤销（除非过期）

场景分析:
- AccessToken 设为 30 分钟过期 → 安全性好，但用户体验差（频繁重新登录）
- AccessToken 设为 30 天过期 → 用户体验好，但安全性差（泄露后长期有效）

解决方案: 双 Token 机制
- AccessToken（短期，如 30 分钟）：用于 API 访问
- RefreshToken（长期，如 7 天）：用于获取新的 AccessToken
```

### 6.2 Refresh Token 流程

```mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务器
    participant DB as 数据库/Redis

    Note over C,DB: === 初次登录 ===
    C->>S: POST /login {username, password}
    S->>DB: 验证凭据
    DB-->>S: 验证通过
    S->>DB: 存储 RefreshToken
    S-->>C: {accessToken, refreshToken, expiresIn}

    Note over C,DB: === 正常访问（30分钟内）===
    C->>S: GET /api/data<br/>Authorization: Bearer accessToken
    S-->>C: 200 OK

    Note over C,DB: === AccessToken 过期后 ===
    C->>S: POST /refresh {refreshToken}
    S->>DB: 查询并验证 RefreshToken
    DB-->>S: 有效
    S->>DB: 生成新 RefreshToken（轮换），旧Token标记为已使用
    S-->>C: {newAccessToken, newRefreshToken}
    C->>S: GET /api/data<br/>Authorization: Bearer newAccessToken
    S-->>C: 200 OK ✅
```

### 6.3 Refresh Token 实现代码框架

```csharp
public class RefreshTokenService
{
    private readonly AppDbContext _dbContext;
    private readonly JwtTokenService _jwtTokenService;

    public class TokenPair
    {
        public string AccessToken { get; set; } = string.Empty;
        public string RefreshToken { get; set; } = string.Empty;
        public int AccessTokenExpiresIn { get; set; }
        public int RefreshTokenExpiresInDays { get; set; }
    }

    /// <summary>
    /// 生成 Token 对
    /// </summary>
    public async Task<TokenPair> GenerateTokenPairAsync(User user, string ipAddress)
    {
        // 1. 生成 AccessToken（短期）
        var userInfo = new UserInfo { /* ... */ };
        var accessToken = _jwtTokenService.GenerateToken(userInfo);

        // 2. 生成 RefreshToken（长期，随机字符串）
        var refreshToken = GenerateSecureRandomToken();
        var refreshTokenHash = HashToken(refreshToken);  // 存储哈希值，不存明文

        // 3. 保存到数据库
        var refreshTokenEntity = new RefreshToken
        {
            TokenHash = refreshTokenHash,
            UserId = user.Id,
            CreatedAt = DateTime.UtcNow,
            ExpiresAt = DateTime.UtcNow.AddDays(7),
            CreatedByIp = ipAddress,
            IsRevoked = false,
            IsUsed = false
        };
        _dbContext.RefreshTokens.Add(refreshTokenEntity);
        await _dbContext.SaveChangesAsync();

        return new TokenPair
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken,
            AccessTokenExpiresIn = 1800,  // 30 分钟
            RefreshTokenExpiresInDays = 7
        };
    }

    /// <summary>
    /// 使用 RefreshToken 获取新的 Token 对
    /// </summary>
    public async Task<TokenPair?> RefreshAsync(string refreshToken, string ipAddress)
    {
        var tokenHash = HashToken(refreshToken);

        // 查找有效的 RefreshToken
        var storedToken = await _dbContext.RefreshTokens
            .Include(rt => rt.User)
            .FirstOrDefaultAsync(rt => rt.TokenHash == tokenHash &&
                                       !rt.IsRevoked &&
                                       !rt.IsUsed &&
                                       rt.ExpiresAt > DateTime.UtcNow);

        if (storedToken == null) return null;

        // 安全检测：如果这个 Token 的家族中有被撤销的 Token，
        // 说明可能存在 Token 泄露，应该撤销整个家族
        var familyTokens = await _dbContext.RefreshTokens
            .Where(rt => rt.UserId == storedToken.UserId &&
                        rt.TokenFamily == storedToken.TokenFamily)
            .ToListAsync();

        if (familyTokens.Any(t => t.IsRevoked))
        {
            // 撤销整个家族的所有 Token
            foreach (var token in familyTokens)
            {
                token.IsRevoked = true;
            }
            await _dbContext.SaveChangesAsync();
            return null;  // 拒绝刷新
        }

        // 标记当前 Token 为已使用
        storedToken.IsUsed = true;
        storedToken.ReplacedByToken = "";  // 新 Token 的哈希值

        // 生成新的 Token 对（Token 轮换）
        var newTokenPair = await GenerateTokenPairAsync(storedToken.User, ipAddress);
        newTokenPair.RefreshToken.ExpiresAt = DateTime.UtcNow.AddDays(7);

        await _dbContext.SaveChangesAsync();
        return newTokenPair;
    }

    private static string GenerateSecureRandomToken()
    {
        using var rng = RandomNumberGenerator.Create();
        var bytes = new byte[64];
        rng.GetBytes(bytes);
        return Convert.ToBase64String(bytes);
    }

    private static string HashToken(string token)
    {
        using var sha = SHA256.Create();
        var bytes = sha.ComputeHash(Encoding.UTF8.GetBytes(token));
        return Convert.ToBase64String(bytes);
    }
}
```

---

## 七、JWT 安全性最佳实践

### 7.1 SecretKey 管理

```csharp
// ❌ 错误做法：硬编码在代码中
var key = "my-secret-key-12345";

// ❌ 错误做法：明文写在配置文件中提交到 Git
// appsettings.json: { "Jwt": { "SecretKey": "plaintext-key" } }

// ✅ 正确做法：从环境变量读取
var key = Environment.GetEnvironmentVariable("JWT_SECRET_KEY")
    ?? throw new Exception("JWT_SECRET_KEY environment variable is required");

// ✅ 正确做法：使用用户机密（开发环境）
// dotnet user-secrets set "Jwt:SecretKey" "dev-secret-key"

// ✅ 正确做法：Azure Key Vault / AWS Secrets Manager（生产环境）
// builder.Services.Configure<KeyVaultOptions>(configuration.GetSection("KeyVault"));
```

### 7.2 安全清单

#### DO ✅

- **DO** 使用足够长且随机的 SecretKey（至少 256 位 / 32 字符）
- **DO** 始终使用 HTTPS 传输 Token
- **DO** 设置合理的过期时间（AccessToken 建议 15-60 分钟）
- **DO** 在 Payload 中添加 jti（JWT ID）防止重放攻击
- **DO** 使用 Refresh Token 机制减少长期 Token 的风险
- **DO** 实现 Token 黑名单机制（如需要即时注销功能）
- **DO** 对敏感操作要求重新认证（即使 Token 未过期）
- **DO** 监控异常的 Token 使用模式

#### DON'T ❌

- **DON'T** 不要在 Payload 中放敏感数据（密码、隐私信息等）
- **DON'T** 不要使用弱签名算法（如 None、HS1）
- **DON'T** 不要在前端 localStorage 中存储敏感 Token（XSS 风险）
- **DON'T** 不要在 URL 中传递 Token（会被服务器日志记录）
- **DON'T** 不要忽略 Token 过期时间的验证
- **DON'T** 不要多个服务共用同一个 SecretKey（应各自独立）
- **DON'T** 不要忘记处理 Token 过期的优雅降级
- **DON'T** 不要信任来自客户端的任何 Claims（必须由服务端签发）

### 7.3 常见安全问题及应对

| 安全威胁 | 描述 | 应对措施 |
|----------|------|----------|
| **Token 劫持** | 通过 XSS 窃取 Token | 使用 HttpOnly Cookie 存储 + CSRF 防护；或使用 Short-lived Token + Refresh Token |
| **算法混淆攻击** | 将算法从 RS256 改为 none | 强制指定算法，拒绝 `none` 算法 |
| **密钥泄露** | SecretKey 被获取 | 定期轮换密钥；使用安全的密钥管理方案 |
| **时钟偏移攻击** | 利用 nbf/exp 的时间差 | 设置合理的 ClockSkew；不要依赖 Token 时间做关键业务判断 |
| **Token 重放** | 截获 Token 后重复使用 | 使用 jti + 黑名单机制；限制单设备在线数 |

---

## 八、练习题

### 练习 1：JWT 解码实战
给定以下 JWT Token，手动解析出其中的 Header、Payload 内容（不需要验证签名）：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IuW8oOS4iSIsJmlhdCI6MTcwNTMxMjgwMH0.xxxxx
```
列出你能识别出的所有 Claims 及其含义。

### 练习 2：配置审查
以下 JWT 配置存在哪些安全隐患？逐一指出并修复：
```csharp
options.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuerSigningKey = false,
    ValidateIssuer = false,
    ValidateAudience = false,
    ValidateLifetime = false,
    ClockSkew = TimeSpan.FromDays(365)
};
```

### 练习 3：双 Token 实现
基于文中提供的 RefreshTokenService 框架，完成以下功能：
- 实现完整的 RefreshToken 数据库实体
- 完成 RefreshAsync 方法的完整实现
- 在 AuthController 中添加 refresh 端点

### 练习 4：权限细化
在 ArticlesController 中实现以下权限控制：
- 普通用户：只能查看自己发布的文章
- 编辑者：可以查看和编辑所有文章
- 管理员：拥有全部权限（包括删除）

### 练习 5：安全加固
为一个已有的 JWT 认证系统设计安全加固方案，包括：
- Token 黑名单机制（Redis 实现）
- 异常登录检测（IP 变化提醒）
- 密码修改后强制所有 Token 失效

---

## 九、延伸阅读

- [RFC 7519: JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [Microsoft Docs: ASP.NET Core 中的 JWT Bearer 身份验证](https://docs.microsoft.com/zh-cn/aspnet/core/security/authentication/jwt)
- [JWT.io - 在线调试工具](https://jwt.io/)
- [OWASP: JSON Web Token (JWT) Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_Cheat_Sheet.html)
- [Auth0: The Ultimate Guide to Handling JWTs](https://auth0.com/blog/the-ultimate-guide-to-handling-jwts-on-asp-net-core/)

---

> **下一节预告**：我们将学习 **Identity Framework 基础**，了解微软官方的用户管理框架如何简化用户注册、登录、角色管理等功能的开发。
