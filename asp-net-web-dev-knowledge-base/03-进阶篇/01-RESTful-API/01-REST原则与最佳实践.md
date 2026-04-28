# REST原则与最佳实践

> **学习目标**：掌握REST架构的核心约束条件，理解资源导向设计思维，能够设计出符合REST规范的API接口
>
> **前置知识**：HTTP协议基础、MVC模式、JSON数据格式
>
> **预计时长**：60-90分钟

---

## 一、什么是REST？

REST（Representational State Transfer，表现层状态转移）是Roy Fielding博士在2000年的博士论文中提出的一种软件架构风格。它不是一种标准，而是一组架构约束条件和设计原则。

### 1.1 REST的六大约束条件

理解REST，首先要理解它的6个核心约束。只有满足这些约束的API才能被称为"RESTful API"：

```mermaid
graph TD
    A[REST六大约束] --> B[客户端-服务器分离]
    A --> C[无状态]
    A --> D[可缓存]
    A --> E[统一接口]
    A --> F[分层系统]
    A --> G[按需代码]

    B --> B1["客户端负责UI<br/>服务器负责数据"]
    C --> C1["每个请求包含所有信息<br/>服务器不保存会话"]
    D --> D1["响应必须标明是否可缓存"]
    E --> E1["资源标识、自描述消息<br/>HATEOAS、统一操作"]
    F --> F1["客户端不知道是否连接到<br/>代理/负载均衡器"]
    G --> G1["可选约束：服务器可返回<br/>可执行代码(如JavaScript)"]
```

#### 约束一：客户端-服务器（Client-Server）

将用户界面（客户端）与数据存储（服务器）分离。这种分离带来的好处：
- **独立演进**：前端和后端可以分别开发和部署
- **可移植性**：后端可以为多种客户端服务（Web、移动端、桌面应用）
- **可扩展性**：可以单独扩展服务器而不影响客户端

```csharp
// 好的设计：前后端分离，通过API通信
// 客户端（React/Vue/Angular）调用API
GET /api/users/1
// 服务器返回JSON数据，不返回HTML
{
    "id": 1,
    "name": "张三",
    "email": "zhangsan@example.com"
}
```

#### 约束二：无状态（Stateless）

这是REST最重要的约束之一。**服务器不保存任何客户端请求之间的会话状态**。

- 每个请求都必须包含处理该请求所需的全部信息
- 服务器不能依赖之前请求的上下文
- 会话状态由客户端维护（通常通过Token）

```csharp
// ✅ 无状态设计：每次请求都携带认证信息
GET /api/orders HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Accept: application/json

// ❌ 有状态设计：依赖Session/Cookie（不符合REST）
// 第一次请求设置Session，后续请求依赖Session
```

#### 约束三：可缓存（Cacheable）

响应消息必须隐式或显式地定义为可缓存或不可缓存的。合理利用缓存可以：
- 减少带宽消耗
- 降低服务器负载
- 提升客户端响应速度

```csharp
// GET请求默认可缓存
[HttpGet("{id}")]
[ResponseCache(Duration = 3600)] // 缓存1小时
public async Task<ActionResult<User>> GetUser(int id)
{
    var user = await _userService.GetByIdAsync(id);
    return Ok(user);
}

// POST/PUT/PATCH/DELETE默认不可缓存
```

#### 约束四：统一接口（Uniform Interface）

这是REST最核心也最具争议的约束。它包含4个子约束：

| 子约束 | 说明 |
|--------|------|
| 资源的标识 | 每个资源都有唯一URI |
| 通过表述操作资源 | 资源有多个表述形式（JSON/XML） |
| 自描述消息 | 每个消息包含如何处理的信息 |
| HATEOAS | 超媒体作为应用状态引擎 |

#### 约束五：分层系统（Layered System）

客户端不需要知道它是直接连接到终端服务器还是中间的代理/网关。这允许：
- 负载均衡
- 共享缓存
- 安全层（防火墙/WAF）
- API网关

#### 约束六：按需代码（Optional - Code on Demand）（可选）

服务器可以临时扩展或自定义客户端功能，通过传输可执行代码（如JavaScript）。这是一个**可选约束**，大多数RESTful API不实现此约束。

---

## 二、资源导向设计思维

### 2.1 核心理念：一切都是资源

在REST中，**一切皆资源**（Everything is a Resource）。资源是API的核心概念，它可以是：
- 实体对象：用户、订单、商品
- 集合概念：订单列表、用户收藏
- 虚拟资源：搜索结果、计算结果
- 控制资源：执行特定操作的端点

### 2.2 名词 vs 动词

这是REST设计中最重要也最容易犯错的地方：

```mermaid
graph LR
    subgraph "❌ 错误做法（RPC风格）"
        A1["/api/getAllUsers"]
        A2["/api/createUser"]
        A3["/api/deleteUser/1"]
        A4["/api/updateUser/1"]
    end

    subgraph "✅ 正确做法（REST风格）"
        B1["GET /api/users"]
        B2["POST /api/users"]
        B3["DELETE /api/users/1"]
        B4["PUT /api/users/1"]
    end
```

**黄金法则**：URL使用名词复数，用HTTP方法表达动作。

### 2.3 URL设计最佳实践

以下是一个博客系统的完整URL设计示例：

```csharp
// ========== 博客系统 RESTful API 设计 ==========

// 文章资源
GET    /api/articles              // 获取文章列表（支持分页、过滤）
GET    /api/articles/{id}         // 获取单篇文章
POST   /api/articles              // 创建新文章
PUT    /api/articles/{id}         // 全量更新文章
PATCH  /api/articles/{id}         // 部分更新文章（如只改标题）
DELETE /api/articles/{id}         // 删除文章

// 文章子资源
GET    /api/articles/{id}/comments     // 获取文章评论列表
POST   /api/articles/{id}/comments     // 给文章添加评论
GET    /api/articles/{id}/comments/{commentId}  // 获取指定评论

// 用户资源
GET    /api/users                  // 获取用户列表
GET    /api/users/{id}             // 获取用户详情
GET    /api/users/{id}/articles    // 获取用户的文章列表
PUT    /api/users/{id}/password    // 修改密码（特殊操作）

// 搜索和统计（特殊资源）
GET    /api/articles/search?q=keyword   // 搜索文章
GET    /api/statistics                   // 获取统计数据
```

---

## 三、HTTP方法语义化使用

正确使用HTTP方法是RESTful API的基础。每种方法都有明确的语义约定：

### 3.1 HTTP方法速查表

| 方法 | 幂等性 | 安全性 | 语义 | 典型状态码 |
|------|--------|--------|------|-----------|
| GET | 是 | 是 | 获取资源 | 200, 404 |
| POST | 否 | 否 | 创建资源 | 201, 400, 409 |
| PUT | 是 | 否 | 全量替换资源 | 200, 204, 400 |
| PATCH | 否 | 否 | 部分更新资源 | 200, 204, 400 |
| DELETE | 是 | 否 | 删除资源 | 204, 404 |

**关键概念解释**：
- **幂等性**：多次执行与一次执行效果相同（如DELETE同一资源多次，结果是相同的——资源不存在了）
- **安全性**：不改变服务器状态

### 3.2 各方法详细说明

#### GET - 查询资源

```csharp
/// <summary>
/// 获取文章列表（支持分页和过滤）
/// </summary>
/// <param name="pageNumber">页码，从1开始</param>
/// <param name="pageSize">每页数量</param>
/// <param name="categoryId">分类ID（可选）</param>
/// <param name="keyword">搜索关键词（可选）</param>
/// <returns>文章分页列表</returns>
[HttpGet]
public async Task<ActionResult<PagedResult<ArticleDto>>> GetArticles(
    [FromQuery] int pageNumber = 1,
    [FromQuery] int pageSize = 10,
    [FromQuery] int? categoryId = null,
    [FromQuery] string? keyword = null)
{
    var query = new ArticleQuery
    {
        PageNumber = pageNumber,
        PageSize = pageSize,
        CategoryId = categoryId,
        Keyword = keyword
    };

    var result = await _articleService.GetPagedAsync(query);
    return Ok(result);
}
```

#### POST - 创建资源

```csharp
/// <summary>
/// 创建新文章
/// </summary>
/// <param name="createDto">文章创建数据</param>
/// <returns>创建成功的文章（含ID）</returns>
[HttpPost]
public async Task<ActionResult<ArticleDto>> CreateArticle(
    [FromBody] CreateArticleDto createDto)
{
    // 自动模型验证（[ApiController]特性自动完成）
    var article = await _articleService.CreateAsync(createDto);

    // 返回201 Created + Location头
    return CreatedAtAction(
        nameof(GetArticle),
        new { id = article.Id },
        article);
}

// CreateArticleDto 示例
public class CreateArticleDto
{
    [Required(ErrorMessage = "标题不能为空")]
    [MaxLength(200, ErrorMessage = "标题最长200字符")]
    public string Title { get; set; } = string.Empty;

    [Required]
    public string Content { get; set; } = string.Empty;

    public int CategoryId { get; set; }

    public List<string>? Tags { get; set; }
}
```

#### PUT vs PATCH 的区别

```csharp
// PUT: 全量更新 - 必须提供所有必填字段
[HttpPut("{id}")]
public async Task<ActionResult<ArticleDto>> UpdateArticle(
    int id, [FromBody] UpdateArticleDto updateDto)
{
    // updateDto 包含所有字段，未提供的字段会被设为默认值/null
    var article = await _articleService.UpdateAsync(id, updateDto);
    return Ok(article);
}

// PATCH: 部分更新 - 只更新提供的字段
[HttpPatch("{id}")]
public async Task<ActionResult> PartialUpdateArticle(
    int id, [FromBody] JsonPatchDocument<ArticleDto> patchDoc)
{
    var article = await _articleService.GetByIdAsync(id);
    if (article == null)
        return NotFound();

    // 应用部分更新
    patchDoc.ApplyTo(article, ModelState);

    if (!ModelState.IsValid)
        return ValidationProblem(ModelState);

    await _articleService.UpdateAsync(id, article);
    return NoContent();
}
```

#### DELETE - 删除资源

```csharp
/// <summary>
/// 删除文章（软删除）
/// </summary>
/// <param name="id">文章ID</param>
[HttpDelete("{id")]
public async Task<ActionResult> DeleteArticle(int id)
{
    var exists = await _articleService.ExistsAsync(id);
    if (!exists)
        return NotFound(new { message = $"文章 {id} 不存在" });

    await _articleService.DeleteAsync(id); // 软删除

    // 成功删除但无返回内容
    return NoContent(); // 204 No Content
}
```

---

## 四、HTTP状态码规范使用

正确使用HTTP状态码可以让API更专业、更容易被客户端理解。

### 4.1 常用状态码分类

```mermaid
graph TD
    subgraph "2xx 成功"
        S1["200 OK - 请求成功"]
        S2["201 Created - 创建成功"]
        S3["204 No Content - 成功无内容"]
    end

    subgraph "3xx 重定向"
        R1["304 Not Modified - 未修改"]
    end

    subgraph "4xx 客户端错误"
        E1["400 Bad Request - 参数错误"]
        E2["401 Unauthorized - 未认证"]
        E3["403 Forbidden - 无权限"]
        E4["404 Not Found - 资源不存在"]
        E5["409 Conflict - 冲突"]
        E6["422 Unprocessable Entity - 语义错误"]
        E7["429 Too Many Requests - 限流"]
    end

    subgraph "5xx 服务器错误"
        SE1["500 Internal Server Error"]
        SE2["502 Bad Gateway"]
        SE3["503 Service Unavailable"]
    end
```

### 4.2 状态码使用规范详解

```csharp
// ========== 状态码使用示例 ==========

[ApiController]
[Route("api/[controller]")]
public class ArticlesController : ControllerBase
{
    // 200 OK - GET请求成功
    [HttpGet("{id}")]
    public async Task<ActionResult<ArticleDto>> GetArticle(int id)
    {
        var article = await _repo.GetByIdAsync(id);
        if (article == null)
            return NotFound(); // 404
        return Ok(article); // 200
    }

    // 201 Created - POST创建成功，返回Location头
    [HttpPost]
    public async Task<ActionResult<ArticleDto>> Create(CreateArticleDto dto)
    {
        var article = await _service.CreateAsync(dto);
        return CreatedAtAction(nameof(GetArticle), new { id = article.Id }, article);
        // 响应头: Location: /api/articles/42
    }

    // 204 No Content - DELETE/PUT成功但无需返回内容
    [HttpDelete("{id}")]
    public async Task<ActionResult> Delete(int id)
    {
        await _service.DeleteAsync(id);
        return NoContent();
    }

    // 400 Bad Request - 请求参数错误
    [HttpPost]
    public async Task<ActionResult> CreateWithValidation(CreateArticleDto dto)
    {
        // [ApiController]特性会自动验证并返回400
        // 手动验证示例:
        if (string.IsNullOrWhiteSpace(dto.Title))
        {
            return BadRequest(new ErrorResponse
            {
                Code = "INVALID_TITLE",
                Message = "标题不能为空",
                Details = new[] { "Title字段是必填的" }
            });
        }
        // ...
    }

    // 401 Unauthorized - 未认证（需要登录）
    [HttpGet("admin-only")]
    [Authorize]
    public ActionResult AdminOnly()
    {
        return Ok("管理员可见");
    }

    // 403 Forbidden - 已认证但无权限
    [HttpGet("super-admin-only")]
    [Authorize(Roles = "SuperAdmin")]
    public ActionResult SuperAdminOnly()
    {
        return Ok("超级管理员可见");
    }

    // 409 Conflict - 资源冲突（如重复创建）
    [HttpPost]
    public async Task<ActionResult> RegisterUser(RegisterUserDto dto)
    {
        var exists = await _userRepo.ExistsByEmailAsync(dto.Email);
        if (exists)
        {
            return Conflict(new
            {
                code = "EMAIL_EXISTS",
                message = "该邮箱已被注册",
                field = "email"
            });
        }
        // ...
    }
}

// 统一错误响应格式
public class ErrorResponse
{
    public string Code { get; set; } = string.Empty;
    public string Message { get; set; } = string.Empty;
    public object? Details { get; set; }
    public string TraceId { get; set; } = string.Empty; // 用于追踪日志
}
```

### 4.3 DO/DON'T 清单

| 场景 | DO (推荐) | DON'T (避免) |
|------|-----------|-------------|
| 创建成功 | 返回 `201 Created` + Location头 | 返回 `200 OK` |
| 删除成功 | 返回 `204 No Content` | 返回 `200 OK` + 空body |
| 找不到资源 | 返回 `404 Not Found` | 返回 `200 OK` + `{success:false}` |
| 参数验证失败 | 返回 `400 Bad Request` + 详细错误 | 返回 `500` 或 `200` + 错误信息 |
| 无权限访问 | 返回 `403 Forbidden` | 返回 `401` 或 `404` |
| 服务器内部错误 | 返回 `500` 并记录日志 | 返回堆栈信息给客户端 |

---

## 五、版本号管理

API版本控制是企业级应用的必备能力。常见的版本号位置有三种方式：

### 5.1 版本控制策略对比

```mermaid
graph TD
    A[版本控制策略] --> B[URL路径版本化]
    A --> C[Query String版本化]
    A --> D[Header版本化]

    B --> B1["/api/v1/users<br/>✅ 直观易调试<br/>✅ 书签友好<br/>❌ 可能破坏URL层级"]
    C --> C1["/api/users?v=1<br/>✅ 不改变URL结构<br/>❌ 容易被忽略"]
    D --> D1["Header: X-API-Version: 1<br/>✅ URL干净<br/>❌ 测试不方便<br/>❌ 浏览器难测试"]

    style B fill:#e1f5fe
    style C fill:#fff3e0
    style D fill:#fce4ec
```

### 5.2 推荐：URL路径版本化

```csharp
// 在Startup.cs或Program.cs中配置
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // 在响应头返回支持的版本
});

// 控制器中使用版本
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[ApiVersion("2.0")] // 支持多版本
public class UsersController : ControllerBase
{
    [MapToApiVersion("1.0")]
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserV1Dto>>> GetV1()
    {
        // V1版本的逻辑
        var users = await _userService.GetAllV1Async();
        return Ok(users);
    }

    [MapToApiVersion("2.0")]
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserV2Dto>>> GetV2()
    {
        // V2版本的逻辑（可能增加了字段）
        var users = await _userService.GetAllV2Async();
        return Ok(users);
    }
}
```

---

## 六、HATEOAS超媒体概念

### 6.1 什么是HATEOAS？

HATEOAS（Hypermedia As The Engine Of Application State）是REST统一接口约束中最复杂的一个。简单来说：**API响应应该包含下一步可以做什么的链接信息**。

### 6.2 HATEOAS示例

```csharp
// 普通响应（没有HATEOAS）
GET /api/articles/1
{
    "id": 1,
    "title": "RESTful API设计指南",
    "author": "张三"
}

// HATEOAS响应（包含链接）
GET /api/articles/1
{
    "id": 1,
    "title": "RESTful API设计指南",
    "author": "张三",
    "_links": {
        "self": { "href": "/api/v1/articles/1", "method": "GET" },
        "update": { "href": "/api/v1/articles/1", "method": "PUT" },
        "delete": { "href": "/api/v1/articles/1", "method": "DELETE" },
        "comments": { "href": "/api/v1/articles/1/comments", "method": "GET" },
        "author": { "href": "/api/v1/users/5", "method": "GET" }
    }
}

// 分页响应中的HATEOAS
GET /api/articles?page=2&size=10
{
    "data": [...],
    "pagination": {
        "currentPage": 2,
        "totalPages": 10,
        "totalCount": 100,
        "hasNext": true,
        "hasPrevious": true
    },
    "_links": {
        "self": { "href": "/api/v1/articles?page=2&size=10" },
        "first": { "href": "/api/v1/articles?page=1&size=10" },
        "prev": { "href": "/api/v1/articles?page=1&size=10" },
        "next": { "href": "/api/v1/articles?page=3&size=10" },
        "last": { "href": "/api/v1/articles?page=10&size=10" }
    }
}
```

### 6.3 实现HATEOAS辅助类

```csharp
/// <summary>
/// 链接信息
/// </summary>
public class LinkDto
{
    public string Href { get; set; } = string.Empty;
    public string Rel { get; set; } = string.Empty; // 关系类型
    public string Method { get; set; } = "GET";
}

/// <summary>
/// HATEOAS响应包装器
/// </summary>
public class ResourceDto<T>
{
    public T Data { get; set; } = default!;
    public List<LinkDto> Links { get; set; } = new();

    public static ResourceDto<T> Create(T data, string selfUrl,
        IUrlHelper urlHelper, object? routeValues = null)
    {
        return new ResourceDto<T>
        {
            Data = data,
            Links = new List<LinkDto>
            {
                new() { Href = selfUrl, Rel = "self", Method = "GET" }
            }
        };
    }
}

// 使用示例
[HttpGet("{id}")]
public async Task<ActionResult<ResourceDto<ArticleDto>>> GetArticle(int id)
{
    var article = await _service.GetByIdAsync(id);
    if (article == null)
        return NotFound();

    var resource = ResourceDto<ArticleDto>.Create(
        article,
        Url.Action(nameof(GetArticle), new { id })!,
        Url);

    // 添加相关链接
    resource.Links.Add(new LinkDto
    {
        Href = Url.Action(nameof(GetComments), new { articleId = id })!,
        Rel = "comments",
        Method = "GET"
    });

    return Ok(resource);
}
```

> **注意**：HATEOAS是REST的理想目标，但在实际项目中可以根据需求决定实现程度。内部API可以简化实现，公开API建议完整实现。

---

## 七、常见反模式（Anti-Patterns）

### 7.1 反模式清单

```mermaid
graph TD
    subgraph "常见REST反模式"
        A1["🚫 URL中使用动词<br/>/api/getUsers → /api/users"]
        A2["🚫 返回HTML而非JSON<br/>应始终返回JSON"]
        A3["🚫 状态码误用<br/>错误时返回200+error字段"]
        A4["🚫 在URL中传递动作<br/>/api/users/1/activate<br/>→ PUT /api/users/1 {status:'active'}"]
        A5["🚫 忽略HTTP方法语义<br/>用GET做删除操作"]
        A6["🚫 嵌套过深<br/>/api/a/1/b/2/c/3/d/4<br/>不超过3层"]
        A7["🚫 用查询参数做 everything<br/>/api?action=create&type=user"]
        A8["🚫 返回不必要的包装<br/>{data:{...}, success:true, code:200}"]
    end
```

### 7.2 反模式代码示例

```csharp
// ==================== 反模式示例 ====================

// ❌ 反模式1：URL中使用动词
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    // 错误
    [HttpGet("getAllUsers")]      // 应该是 GET /api/users
    [HttpGet("getUserById/{id}")] // 应该是 GET /api/users/{id}
    [HttpPost("createUser")]      // 应该是 POST /api/users
    [HttpPost("updateUser")]      // 应该是 PUT /api/users/{id}
    [HttpPost("deleteUser/{id}")] // 应该是 DELETE /api/users/{id}
}

// ❌ 反模式2：返回HTML视图
public IActionResult Index()
{
    return View(); // 这是MVC控制器做的事，不是API
}

// ❌ 反模式3：总是返回200
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    bool success = _repo.Delete(id);
    // 错误：无论成功失败都返回200
    return Ok(new { success, message = success ? "成功" : "失败" });

    // 正确：成功返回204，失败返回404/500
}

// ❌ 反模式4：过深的嵌套
// /api/companies/1/departments/2/teams/3/members/4/tasks/5
// 建议：扁平化为 /api/tasks/5 或 /api/members/4/tasks/5

// ❌ 反模式5：不必要的数据包装
// 返回 { "code": 200, "message": "success", "data": {...} }
// HTTP本身已经有状态码了，不需要再包一层
```

---

## 八、实战案例：博客系统RESTful API完整设计

### 8.1 项目结构

```
BlogApi/
├── Controllers/
│   ├── ArticlesController.cs       # 文章API
│   ├── CommentsController.cs       # 评论API
│   └── UsersController.cs          # 用户API
├── DTOs/
│   ├── Articles/
│   │   ├── ArticleDto.cs           # 文章详情DTO
│   │   ├── CreateArticleDto.cs     # 创建文章DTO
│   │   └── UpdateArticleDto.cs     # 更新文章DTO
│   └── Common/
│       ├── PagedResult.cs          # 分页结果
│       └── ErrorResponse.cs        # 错误响应
├── Services/
│   ├── IArticleService.cs          # 文章服务接口
│   └── ArticleService.cs           # 文章服务实现
├── Models/
│   └── Entities/
│       ├── Article.cs              # 文章实体
│       └── Comment.cs              # 评论实体
└── Helpers/
    └── LinkGenerator.cs            # HATEOAS链接生成
```

### 8.2 完整控制器示例

```csharp
using Microsoft.AspNetCore.Mvc;
using BlogApi.DTOs.Articles;
using BlogApi.DTOs.Common;
using BlogApi.Services;

namespace BlogApi.Controllers;

/// <summary>
/// 文章管理API
/// </summary>
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/articles")]
[Produces("application/json")]
public class ArticlesController : ControllerBase
{
    private readonly IArticleService _articleService;
    private readonly ILogger<ArticlesController> _logger;

    public ArticlesController(
        IArticleService articleService,
        ILogger<ArticlesController> logger)
    {
        _articleService = articleService;
        _logger = logger;
    }

    /// <summary>
    /// 获取文章分页列表
    /// </summary>
    /// <param name="pageNumber">页码（默认1）</param>
    /// <param name="pageSize">每页数量（默认10，最大100）</param>
    /// <param name="categoryId">分类筛选</param>
    /// <param name="keyword">关键词搜索</param>
    /// <param name="sortBy">排序字段</param>
    /// <param name="sortOrder">排序方向</param>
    /// <returns>文章分页列表</returns>
    /// <response code="200">获取成功</response>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ArticleDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<ArticleDto>>> GetArticles(
        [FromQuery] int pageNumber = 1,
        [FromQuery] int pageSize = 10,
        [FromQuery] int? categoryId = null,
        [FromQuery] string? keyword = null,
        [FromQuery] string sortBy = "createdAt",
        [FromQuery] string sortOrder = "desc")
    {
        // 参数校验
        if (pageNumber < 1)
            return BadRequest(new ErrorResponse("INVALID_PAGE", "页码必须大于0"));

        if (pageSize < 1 || pageSize > 100)
            return BadRequest(new ErrorResponse("INVALID_SIZE", "每页数量必须在1-100之间"));

        try
        {
            var query = new ArticleQuery
            {
                PageNumber = pageNumber,
                PageSize = pageSize,
                CategoryId = categoryId,
                Keyword = keyword,
                SortBy = sortBy,
                SortOrder = sortOrder
            };

            var result = await _articleService.GetPagedAsync(query);
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "获取文章列表失败");
            return StatusCode(500, new ErrorResponse("INTERNAL_ERROR", "服务器内部错误"));
        }
    }

    /// <summary>
    /// 获取文章详情
    /// </summary>
    /// <param name="id">文章ID</param>
    /// <returns>文章详情</returns>
    /// <response code="200">获取成功</response>
    /// <response code="404">文章不存在</response>
    [HttpGet("{id:int}", Name = "GetArticleById")]
    [ProducesResponseType(typeof(ArticleDto), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ErrorResponse), StatusCodes.Status404NotFound)]
    [ResponseCache(Duration = 300, VaryByQueryKeys = new[] { "id" })]
    public async Task<ActionResult<ArticleDto>> GetArticle(int id)
    {
        var article = await _articleService.GetByIdAsync(id);

        if (article is null)
            return NotFound(new ErrorResponse("NOT_FOUND", $"文章 {id} 不存在"));

        return Ok(article);
    }

    /// <summary>
    /// 创建新文章
    /// </summary>
    /// <param name="dto">文章创建数据</param>
    /// <returns>创建的文章</returns>
    /// <response code="201">创建成功</response>
    /// <response code="400">参数验证失败</response>
    /// <response code="401">未认证</response>
    [HttpPost]
    [Authorize]
    [ProducesResponseType(typeof(ArticleDto), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ArticleDto>> CreateArticle([FromBody] CreateArticleDto dto)
    {
        var userId = HttpContext.User.FindFirstValue(ClaimTypes.NameIdentifier)!;

        try
        {
            var article = await _articleService.CreateAsync(dto, userId);

            return CreatedAtAction(
                nameof(GetArticle),
                new { id = article.Id, version = "1.0" },
                article);
        }
        catch (InvalidOperationException ex)
        {
            return Conflict(new ErrorResponse("CONFLICT", ex.Message));
        }
    }

    /// <summary>
    /// 全量更新文章
    /// </summary>
    /// <param name="id">文章ID</param>
    /// <param name="dto">更新的文章数据</param>
    /// <returns>更新后的文章</returns>
    [HttpPut("{id:int}")]
    [Authorize]
    [ProducesResponseType(typeof(ArticleDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(ErrorResponse), StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ArticleDto>> UpdateArticle(int id, [FromBody] UpdateArticleDto dto)
    {
        var exists = await _articleService.ExistsAsync(id);
        if (!exists)
            return NotFound(new ErrorResponse("NOT_FOUND", $"文章 {id} 不存在"));

        var article = await _articleService.UpdateAsync(id, dto);
        return Ok(article);
    }

    /// <summary>
    /// 删除文章（软删除）
    /// </summary>
    /// <param name="id">文章ID</param>
    /// <response code="204">删除成功</response>
    /// <response code="404">文章不存在</response>
    [HttpDelete("{id:int}")]
    [Authorize(Roles = "Admin,Author")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(typeof(ErrorResponse), StatusCodes.Status404NotFound)]
    public async Task<ActionResult> DeleteArticle(int id)
    {
        var exists = await _articleService.ExistsAsync(id);
        if (!exists)
            return NotFound(new ErrorResponse("NOT_FOUND", $"文章 {id} 不存在"));

        await _articleService.SoftDeleteAsync(id);
        return NoContent();
    }
}
```

---

## 九、总结与检查清单

### 9.1 RESTful API设计检查清单

在设计API前，请逐项确认：

- [ ] **URL使用名词复数**：`/api/users` 而非 `/api/user`
- [ ] **使用标准HTTP方法**：GET/POST/PUT/PATCH/DELETE
- [ ] **正确使用状态码**：201表示创建、204表示无内容返回、400表示参数错误
- [ ] **保持无状态**：不在服务器保存会话状态
- [ ] **使用JSON格式**：请求和响应都使用JSON
- [ ] **支持分页**：列表接口必须支持分页
- [ ] **版本控制**：从第一天就考虑版本策略
- [ ] **统一错误格式**：定义标准的错误响应结构
- [ ] **文档完善**：提供完整的API文档（Swagger）
- [ ] **安全考虑**：认证、授权、输入验证、输出脱敏
- [ ] **性能优化**：缓存、压缩、异步操作
- [ ] **URL层级不超过3层**：避免过度嵌套

### 9.2 本节核心要点回顾

| 要点 | 关键内容 |
|------|---------|
| REST核心 | 六大约束：CS分离、无状态、可缓存、统一接口、分层系统、按需代码 |
| 设计理念 | 一切皆资源，名词复数，HTTP方法表动作 |
| HTTP方法 | GET查、POST建、PUT全量改、PATCH部分改、DELETE删 |
| 状态码 | 200成功、201创建、204无内容、400参数错、401未认证、403无权、404不存在、500服务器错 |
| 版本控制 | 推荐URL路径版本化 `/api/v1/resource` |
| HATEOAS | 响应中包含可用操作的链接信息 |

---

## 练习题

### 练习1：URL设计判断
判断以下哪些URL设计符合REST规范？说明原因。
1. `POST /api/login`
2. `GET /api/users/active`
3. `DELETE /api/users/1/profile`
4. `PUT /api/users/1/status`
5. `GET /api/search?q=rest`

### 练习2：选择正确的HTTP方法和状态码
为以下场景选择合适的HTTP方法和返回状态码：
1. 用户注册
2. 获取当前登录用户信息
3. 修改用户昵称
4. 取消订单
5. 上传头像

### 练习3：设计一个电商系统的商品API
要求包含：商品列表（分页、筛选、排序）、商品详情、创建商品、更新商品、删除商品、商品上下架。

### 练习4：分析反模式
以下代码有什么问题？请指出并修正：
```csharp
[HttpPost("getProductList")]
public IActionResult GetProducts() => Ok(_repo.GetAll());
```

### 练习5：HATEOAS实践
为一个用户资源设计包含HATEOAS链接的响应格式，至少包含self、orders、profile三个链接。

---

### 参考答案

**练习1答案**：
1. **不完全符合**：login是一个动作，但这是特例——认证通常作为独立端点是可以接受的，或者可以用 `POST /api/auth/token` 或 `POST /api/sessions`
2. **符合**：获取活跃用户列表，active是过滤条件
3. **有问题**：删除profile应该用 `DELETE /api/users/1` 或 `PATCH /api/users/1` 移除profile字段
4. **符合**：PUT更新资源的status属性
5. **符合**：search作为特殊资源是可以接受的

**练习2答案**：
1. `POST /api/auth/register` -> 201 Created
2. `GET /api/users/me` -> 200 OK
3. `PATCH /api/users/1` body:`{"nickname":"新昵称"}` -> 200 OK
4. `DELETE /api/orders/1` 或 `PATCH /api/orders/1` body:`{"status":"cancelled"}` -> 200/204
5. `POST /api/users/1/avatar` (multipart/form-data) 或 `PUT /api/users/1` body包含avatar -> 200/201

**练习3答案**：
```
GET    /api/v1/products?page=1&size=20&category=electronics&sort=price&order=asc
GET    /api/v1/products/{id}
POST   /api/v1/products
PUT    /api/v1/products/{id}
PATCH  /api/v1/products/{id}
DELETE /api/v1/products/{id}
PATCH  /api/v1/products/{id}/status  body:{"status":"on/off"}
```

**练习4答案**：
问题：
1. URL用了动词getProductList，应该是 `GET /api/products`
2. POST用于读取操作，应该用GET
3. 没有分页支持

修正：
```csharp
[HttpGet]
public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
    [FromQuery] int page = 1, [FromQuery] int size = 20)
{
    var result = await _repo.GetPagedAsync(page, size);
    return Ok(result);
}
```

**练习5答案**：
```json
{
    "id": 1,
    "name": "张三",
    "email": "zhangsan@example.com",
    "_links": {
        "self": { "href": "/api/v1/users/1", "method": "GET" },
        "orders": { "href": "/api/v1/users/1/orders", "method": "GET" },
        "profile": { "href": "/api/v1/users/1/profile", "method": "GET" },
        "update": { "href": "/api/v1/users/1", "method": "PUT" },
        "avatar": { "href": "/api/v1/users/1/avatar", "method": "PUT" }
    }
}
```

---

## 延伸阅读

- [Roy Fielding的REST论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) - REST架构风格的原始定义
- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md) - 微软的API设计指南
- [Zalando RESTful API Guidelines](https://opensource.zalando.com/restful-api-guidelines/) - 业界知名的RESTful API设计规范
- [Google API Design Guide](https://cloud.google.com/apis/design/) - Google的API设计最佳实践

---

## 上下节导航

- **上一节**：[Entity Framework Core进阶](../02-EF-Core-进阶/README.md)
- **下一节**：[API控制器开发](02-API控制器开发.md) - 学习如何在ASP.NET Core中高效开发API控制器
