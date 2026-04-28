# Todo 待办事项管理应用

> **对应教程**：[[基础篇/04-实战项目-Todo应用]]
> **难度等级**：⭐⭐⭐ 中级 | **预计耗时**：5-8小时
> **适用人群**：已完成 Hello World 项目，希望掌握 CRUD + EF Core + DI 的开发者

---

## 项目概述

这是一个**功能完整的 CRUD 待办事项管理系统**，涵盖了 ASP.NET Core MVC 开发的核心流程：从数据库设计、Entity Framework Core ORM 映射、依赖注入架构，到表单验证、分页查询、单元测试和部署准备。

本项目不仅是一个简单的增删改查示例，它展示了**企业级应用的基础架构模式**（Controller → Service → Repository → Data 四层分离），并融入了性能优化、异常处理、测试驱动开发等最佳实践。通过构建这个项目，你将获得从零搭建真实 Web 应用的完整经验。

**核心价值**：
- 掌握 EF Core Code First 数据库开发流程
- 理解依赖注入（DI）和 inversion of control（IoC）的实际应用
- 学会设计可测试的分层架构
- 建立生产级代码质量意识（验证、异常处理、性能优化）

---

## 技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **ASP.NET Core MVC** | 8.0 | Web 应用框架（Model-View-Controller 模式） |
| **Entity Framework Core** | 8.0 | ORM 框架，对象关系映射与数据库操作 |
| **SQL Server** | 2019+ / LocalDB | 关系型数据库存储 |
| **xUnit** | 2.6.x | 单元测试框架 |
| **Moq** | 4.20.x | Mock 框架，用于隔离外部依赖 |
| **Bootstrap 5** | 5.3.x | 前端 UI 框架 |
| **jQuery** | 3.7.x | DOM 操作和 AJAX 请求 |
| **C#** | 12 | 编程语言 |

**为什么选择这个技术组合？**
- MVC 模式是学习服务端开发的经典起点，比 Web API 更直观
- EF Core 是 .NET 生态最成熟的 ORM，掌握它等于掌握了数据访问层的半壁江山
- SQL Server 与 .NET 天然集成，LocalDB 让本地开发零配置
- xUnit + Moq 是 .NET 单元测试的黄金搭档

---

## 功能清单

### 核心功能模块（9大模块）

#### 1️⃣ 用户注册登录系统（Cookie 认证）
- **注册功能**：
  - 用户名/邮箱/密码/确认密码字段
  - 密码强度验证（最少8位，包含大小写字母和数字）
  - 邮箱格式验证
  - 用户名唯一性检查（Remote 验证）
  - 密码哈希存储（ASP.NET Core Identity 或自定义实现）
  
- **登录功能**：
  - 用户名或邮箱登录（双字段支持）
  - Cookie-based 会话管理（Authentication Cookie）
  - "记住我"选项（Persistent Cookie）
  - 登录失败次数限制（防暴力破解）
  - 登录后重定向到原始请求页面

- **登出功能**：
  - 清除认证 Cookie
  - 重定向到首页

- **密码相关**：
  - 忘记密码流程（可选，发送重置链接）
  - 修改密码（需验证旧密码）

**涉及文件**：
- `Controllers/AccountController.cs` — 认证逻辑
- `Views/Account/Login.cshtml` / `Register.cshtml` — 认证视图
- `Models/AccountViewModels.cs` — 登录/注册模型
- `Services/IUserService.cs` / `UserService.cs` — 用户业务逻辑

---

#### 2️⃣ Todo CRUD 完整操作
- **创建 Todo（Create）**：
  - 表单页面：标题（必填）、描述（可选）、截止日期、优先级（低/中/高）、标签（多选）
  - 服务端验证 + 客户端实时验证
  - 创建成功后跳转到列表页并显示成功消息
  
- **读取 Todo（Read）**：
  - 列表页：表格展示所有 Todos（分页显示）
  - 详情页：单个 Todo 的完整信息展示
  - 支持按状态筛选（全部/进行中/已完成/已过期）

- **更新 Todo（Update）**：
  - 编辑页面：预填充当前值
  - 标记完成/取消完成的快捷操作
  - 更新时间戳自动记录（UpdatedAt 字段）
  - 并发控制（乐观锁，RowVersion）

- **删除 Todo（Delete）**：
  - 软删除（IsDeleted 标记，不物理删除）
  - 硬删除选项（管理员权限）
  - 删除前二次确认对话框
  - 批量删除（勾选多个 → 一键删除）

- **批量操作**：
  - 批量标记完成
  - 批量更改优先级
  - 批量分配标签
  - 批量导出（CSV/Excel 格式）

**涉及文件**：
- `Controllers/TodoController.cs` — CRUD 操作
- `Views/Todo/Index.cshtml` / `Create.cshtml` / `Edit.cshtml` / `Details.cshtml`
- `Models/TodoItem.cs` / `TodoViewModels.cs`
- `Services/ITodoService.cs` / `TodoService.cs`

---

#### 3️⃣ 表单验证体系
本项目实现了**三层验证机制**：

**第一层：Data Annotations（声明式验证）**
```csharp
public class CreateTodoViewModel
{
    [Required(ErrorMessage = "标题不能为空")]
    [StringLength(200, MinimumLength = 2, ErrorMessage = "标题长度必须在2-200个字符之间")]
    public string Title { get; set; } = string.Empty;
    
    [DataType(DataType.Date)]
    [FutureDateValidator(ErrorMessage = "截止日期必须是未来的日期")]
    public DateTime? DueDate { get; set; }
    
    [Range(1, 3, ErrorMessage = "优先级必须在1-3之间")]
    public Priority Priority { get; set; } = Priority.Medium;
}
```

**第二层：自定义验证器（Custom Validator）**
```csharp
// 自定义属性：验证日期不能是过去的日期
public class FutureDateValidatorAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value is DateTime date && date < DateTime.Today)
        {
            return new ValidationResult(ErrorMessage ?? "日期不能是过去的时间");
        }
        return ValidationResult.Success!;
    }
}
```

**第三层：Remote 验证（AJAX 异步校验）**
```csharp
// 控制器中的 Remote Action
[AcceptVerbs("GET", "POST")]
public IActionResult VerifyTitle(string title)
{
    if (_todoService.ExistsByTitle(title))
    {
        return Json($"标题 '{title}' 已被使用");
    }
    return Json(true);
}

// 模型中的应用
[Remote(action: "VerifyTitle", controller: "Todo", ErrorMessage = "标题已存在")]
public string Title { get; set; }
```

**客户端验证集成**：
- jQuery Validation 插件自动解析 Data Annotations
- 实时反馈（输入时即验证，无需提交表单）
- 自定义错误消息样式（Bootstrap 验证样式）

**涉及文件**：
- `Models/TodoViewModels.cs` — 验证规则定义
- `Attributes/FutureDateValidatorAttribute.cs` — 自定义验证器
- `Validators/` — 复杂业务验证逻辑
- `wwwroot/js/validation.js` — 客户端验证脚本

---

#### 4️⃣ 分页查询系统
使用泛型分页组件 `PagedResult<T>` 实现高效的数据分页：

**PagedResult<T> 泛型类**：
```csharp
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; } = Enumerable.Empty<T>();
    public int CurrentPage { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling((double)TotalCount / PageSize);
    public bool HasPrevious => CurrentPage > 1;
    public bool HasNext => CurrentPage < TotalPages;
}
```

**Repository 层分页实现**：
```csharp
public async Task<PagedResult<TodoItem>> GetPagedAsync(int page, int pageSize, 
    Func<IQueryable<TodoItem>, IOrderedQueryable<TodoItem>>? orderBy = null)
{
    var query = _context.Todos
        .Where(t => !t.IsDeleted)
        .AsNoTracking(); // 性能优化：不跟踪实体变化
    
    var totalCount = await query.CountAsync();
    
    if (orderBy != null)
        query = orderBy(query);
    else
        query = query.OrderByDescending(t => t.CreatedAt);
    
    var items = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return new PagedResult<TodoItem>
    {
        Items = items,
        CurrentPage = page,
        PageSize = pageSize,
        TotalCount = totalCount
    };
}
```

**View 层分页组件（TagHelper）**：
```html
<!-- 使用自定义 TagHelper 渲染分页导航 -->
<pagination info="Model.PagedInfo" 
            page-action="Index" 
            page-controller="Todo"
            class-pagination="pagination justify-content-center">
</pagination>
```

**特性**：
- 支持 URL 参数传递页码（?page=2&pageSize=10）
- 可配置每页显示数量（5/10/20/50）
- 显示总记录数和当前页码信息
- 首页/上一页/下一页/末页快捷按钮
- 高亮当前页码

**涉及文件**：
- `Common/PagedResult.cs` — 泛型分页结果类
- `Repositories/TodoRepository.cs` — 分页查询实现
- `TagHelpers/PaginationTagHelper.cs` — 分页 TagHelper
- `Views/Shared/Components/Pagination/Default.cshtml` — 分页视图组件

---

#### 5️⃣ 搜索过滤排序
提供多维度的数据检索能力：

**搜索功能**：
- 关键词搜索（标题 + 描述内容模糊匹配）
- 即时搜索（输入即搜，500ms 防抖延迟）
- 搜索历史记录（LocalStorage 存储，最多保存 10 条）

**过滤功能**：
- 按状态过滤（全部/待办/进行中/已完成/已过期）
- 按优先级过滤（低/中/高）
- 按标签过滤（多选）
- 按日期范围过滤（创建日期/截止日期）
- 组合过滤（AND 逻辑，可叠加多个条件）

**排序功能**：
- 支持多字段排序（创建时间/截止日期/优先级/标题）
- 升序/降序切换
- 点击列头即可排序（带箭头指示器）
- 默认按创建时间降序排列

**URL 状态同步**：
所有搜索、过滤、排序参数都同步到 URL QueryString，支持：
- 书签收藏特定查询条件
- 浏览器前进/后退按钮保持状态
- 分享搜索结果链接给他人

示例 URL：
```
/Todo?search=报告&page=2&status=Pending&priority=High&sortBy=DueDate&sortOrder=desc&tags=工作,紧急
```

**涉及文件**：
- `Models/TodoSearchFilter.cs` — 搜索过滤器模型
- `Services/ITodoService.cs` — 搜索方法签名
- `Views/Todo/Index.cshtml` — 搜索栏 UI
- `wwwroot/js/search.js` — 前端搜索逻辑

---

#### 6️⃣ 全局异常处理
建立完善的错误处理机制：

**三层异常捕获**：

1. **全局异常中间件**（`ExceptionMiddleware.cs`）：
   ```csharp
   public class ExceptionMiddleware
   {
       private readonly RequestDelegate _next;
       private readonly ILogger<ExceptionMiddleware> _logger;
       private readonly IHostEnvironment _env;

       public async Task InvokeAsync(HttpContext context)
       {
           try
           {
               await _next(context);
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "未处理的异常: {Message}", ex.Message);
               await HandleExceptionAsync(context, ex);
           }
       }

       private async Task HandleExceptionAsync(HttpContext context, Exception exception)
       {
           context.Response.ContentType = "application/json";
           
           var response = context.Response;
           var errorResponse = new ErrorResponse();

           switch (exception)
           {
               case DbUpdateException dbEx:
                   response.StatusCode = (int)HttpStatusCode.Conflict;
                   errorResponse.Message = "数据库操作失败";
                   errorResponse.Details = _env.IsDevelopment() ? dbEx.InnerException?.Message : null;
                   break;
                   
               case ValidationException valEx:
                   response.StatusCode = (int)HttpStatusCode.BadRequest;
                   errorResponse.Message = "验证失败";
                   errorResponse.Errors = valEx.Errors;
                   break;
                   
               case NotFoundException notFoundEx:
                   response.StatusCode = (int)HttpStatusCode.NotFound;
                   errorResponse.Message = notFoundEx.Message;
                   break;
                   
               default:
                   response.StatusCode = (int)HttpStatusCode.InternalServerError;
                   errorResponse.Message = "服务器内部错误";
                   errorResponse.Details = _env.IsDevelopment() ? exception.StackTrace : null;
                   break;
           }

           await context.Response.WriteAsync(JsonSerializer.Serialize(errorResponse));
       }
   }
   ```

2. **Controller 层 `[HandleError]` 特性**：
   ```csharp
   [HandleException(ExceptionType = typeof(NotFoundException), View = "NotFound")]
   public IActionResult Details(int id)
   {
       // ...
   }
   ```

3. **Global Filter**（Program.cs 注册）：
   ```csharp
   builder.Services.AddControllersWithViews(options =>
   {
       options.Filters.Add<GlobalExceptionFilter>();
   });
   ```

**自定义异常类型**：
- `NotFoundException` — 资源不存在（404）
- `ValidationException` — 业务验证失败（400）
- `UnauthorizedAccessException` — 未授权访问（401）
- `ForbiddenException` — 权限不足（403）
- `ConflictException` — 资源冲突（409）

**用户友好错误页**：
- `Views/Shared/Error.cshtml` — 通用错误页
- `Views/Shared/NotFound.cshtml` — 404 页面
- `Views/Shared/AccessDenied.cshtml` — 403 页面

**日志记录**：
- 使用 ILogger 记录异常详情（级别：Error）
- 包含请求路径、用户 ID、时间戳等上下文信息
- 生产环境隐藏敏感堆栈信息

**涉及文件**：
- `Middleware/ExceptionMiddleware.cs` — 全局异常中间件
- `Filters/GlobalExceptionFilter.cs` — 全局异常过滤器
- `Exceptions/` — 自定义异常类目录
- `Views/Shared/Error.cshtml` — 错误视图
- `Program.cs` — 中间件注册

---

#### 7️⃣ 性能优化策略
本项目集成了多种性能优化技术：

**AsNoTracking（只读查询优化）**：
```csharp
// 场景：列表页查询不需要跟踪实体变化
var todos = await _context.Todos
    .AsNoTracking() // 不创建变更追踪代理，减少内存占用
    .Where(t => t.UserId == userId)
    .ToListAsync();
```
**效果**：查询性能提升 30-50%，内存占用降低

**Select 投影（减少数据传输）**：
```csharp
// 只查询需要的字段，避免 SELECT *
public async Task<IEnumerable<TodoListDto>> GetListAsync(int userId)
{
    return await _context.Todos
        .Where(t => t.UserId == userId && !t.IsDeleted)
        .Select(t => new TodoListDto  // 投影到 DTO
        {
            Id = t.Id,
            Title = t.Title,
            Priority = t.Priority,
            Status = t.Status,
            DueDate = t.DueDate,
            CreatedAt = t.CreatedAt
            // 注意：不包含 Description（可能很长）
        })
        .ToListAsync();
}
```
**效果**：减少网络传输量 60-80%

**响应压缩（Brotli/Gzip）**：
```csharp
// Program.cs 配置
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[]
    {
        "application/json",
        "text/plain"
    });
});
app.UseResponseCompression(); // 必须在 UseRouting() 之前
```
**效果**：HTML/CSS/JS 文件体积减小 60-80%

**其他优化措施**：
- **EF Core 查询优化**：使用 `.Include()` 避免 N+1 问题
- **异步编程**：所有 I/O 操作使用 `async/await`
- **连接池管理**：EF Core 默认启用连接池
- **静态资源缓存**：配置 Cache-Control 头
- **图片懒加载**：只加载可视区域内的图片
- **最小化 CSS/JS**：生产环境使用压缩版本

**性能指标参考**：
- 首页加载时间：< 1秒（局域网）/ < 3秒（公网）
- 数据库查询时间：< 100ms（10000 条记录以内）
- 并发支持：100+ 同时在线用户（单服务器）

**涉及文件**：
- `Repositories/BaseRepository.cs` — 基础 Repository（含 AsNoTracking）
- `Services/TodoService.cs` — Select 投影使用
- `Program.cs` — 压缩中间件配置
- `DTOs/TodoListDto.cs` — 数据传输对象

---

#### 8️⃣ 单元测试（xUnit + Moq）
完整的测试覆盖，确保代码质量：

**测试框架选择理由**：
- **xUnit**：.NET 社区最流行的测试框架，与 Visual Studio 深度集成
- **Moq**：强大的 Mock 框架，可以模拟任何接口或抽象类

**测试项目结构**：
```
TodoApp.Tests/
├── TodoApp.Tests.csproj
├── Controllers/
│   ├── TodoControllerTests.cs      # Controller 测试
│   └── AccountControllerTests.cs  # 认证控制器测试
├── Services/
│   └── TodoServiceTests.cs         # Service 层测试
├── Repositories/
│   └── TodoRepositoryTests.cs     # Repository 层测试
├── Helpers/
│   ├── TestDataGenerator.cs        # 测试数据生成器
│   └── MockHelpers.cs             # Mock 工具类
└── appsettings.json                # 测试配置
```

**9 个核心测试用例**：

| # | 测试名称 | 类型 | 验证点 |
|---|---------|------|--------|
| 1 | `TodoController_Index_ReturnsViewWithPagedResult` | Controller | 首页正确返回分页数据 |
| 2 | `TodoController_Details_WithValidId_ReturnsView` | Controller | 详情页正确显示 Todo 信息 |
| 3 | `TodoController_Details_WithInvalidId_ReturnsNotFound` | Controller | 无效 ID 返回 404 |
| 4 | `TodoController_Create_Post_ValidModel_RedirectsToIndex` | Controller | 创建成功后重定向到列表 |
| 5 | `TodoController_Create_Post_InvalidModel_ReturnsViewWithErrors` | Controller | 验证失败返回表单及错误信息 |
| 6 | `TodoController_Delete_Post_ValidId_DeletesAndRedirects` | Controller | 删除成功并重定向 |
| 7 | `TodoService_CreateAsync_ValidData_CreatesTodo` | Service | Service 层创建逻辑正确 |
| 8 | `TodoService_GetPagedAsync_ReturnsCorrectPage` | Service | 分页逻辑正确（页码、总数） |
| 9 | `TodoService_DeleteAsync_NonExistingId_ThrowsException` | Service | 删除不存在的数据抛出异常 |

**测试示例代码**：
```csharp
public class TodoControllerTests
{
    private readonly Mock<ITodoService> _mockTodoService;
    private readonly Mock<IUserService> _mockUserService;
    private readonly TodoController _controller;

    public TodoControllerTests()
    {
        _mockTodoService = new Mock<ITodoService>();
        _mockUserService = new Mock<IUserService>();
        _controller = new TodoController(_mockTodoService.Object, _mockUserService.Object);
    }

    [Fact]
    public async Task Index_ReturnsViewWithPagedResult()
    {
        // Arrange（准备）
        var pagedResult = new PagedResult<TodoDto>
        {
            Items = new List<TodoDto> { new() { Id = 1, Title = "Test Todo" } },
            CurrentPage = 1,
            PageSize = 10,
            TotalCount = 1
        };
        _mockTodoService.Setup(s => s.GetPagedAsync(1, 10, It.IsAny<string>(), It.IsAny<TodoStatus?>()))
            .ReturnsAsync(pagedResult);

        // Act（执行）
        var result = await _controller.Index();

        // Assert（断言）
        var viewResult = Assert.IsType<ViewResult>(result);
        Assert.IsAssignableFrom<PagedResult<TodoDto>>(viewResult.Model);
        _mockTodoService.Verify(s => s.GetPagedAsync(1, 10, It.IsAny<string>(), It.IsAny<TodoStatus?>()), Times.Once);
    }

    [Fact]
    public async Task Details_WithValidId_ReturnsViewWithTodo()
    {
        // Arrange
        var todo = new TodoDto { Id = 1, Title = "Test", Description = "Test desc" };
        _mockTodoService.Setup(s => s.GetByIdAsync(1)).ReturnsAsync(todo);

        // Act
        var result = await _controller.Details(1);

        // Assert
        var viewResult = Assert.IsType<ViewResult>(result);
        Assert.Equal(todo, viewResult.Model);
    }
}
```

**运行测试**：
```bash
# 运行所有测试
dotnet test

# 运行指定测试类的测试
dotnet test --filter "FullyQualifiedName~TodoControllerTests"

# 运行单个测试方法
dotnet test --filter "FullyQualifiedName~TodoControllerTests.Index_ReturnsViewWithPagedResult"

# 生成测试覆盖率报告
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

**目标覆盖率**：
- Controller 层：≥ 80%
- Service 层：≥ 90%
- Repository 层：≥ 85%
- 整体代码覆盖率：≥ 85%

**涉及文件**：
- `TodoApp.Tests/TodoApp.Tests.csproj` — 测试项目配置
- `TodoApp.Tests/Controllers/TodoControllerTests.cs` — Controller 测试
- `TodoApp.Tests/Services/TodoServiceTests.cs` — Service 测试
- `TodoApp.Tests/Helpers/MockHelpers.cs` — Mock 辅助工具

---

#### 9️⃣ 部署准备方案
提供两种生产环境部署方案：

**方案 A：IIS 部署（传统 Windows 方案）**

**前置要求**：
- Windows Server 2019+ / Windows 10/11
- IIS 10.0+
- ASP.NET Core Hosting Bundle（运行时安装包）

**部署步骤**：
1. **发布项目**：
   ```bash
   dotnet publish -c Release -o ./publish
   ```

2. **安装 Hosting Bundle**：
   - 下载并安装 [.NET 8.0 Hosting Bundle](https://dotnet.microsoft.com/download/dotnet/8.0)
   - 安装完成后重启 IIS（`iisreset`）

3. **配置 IIS**：
   - 打开 IIS Manager
   - 创建新网站（站点名称、物理路径指向 publish 目录、端口 80/443）
   - 配置应用程序池：.NET CLR 版本设为"无托管代码"
   - 设置权限：IIS_IUSRS 和 IUSR 对发布目录有读取权限

4. **配置 web.config**（IIS 配置文件，publish 目录下自动生成）：
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
     <system.webServer>
       <handlers>
         <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
       </handlers>
       <aspNetCore processPath="dotnet" arguments=".\TodoApp.Web.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
     </system.webServer>
   </configuration>
   ```

5. **配置数据库连接字符串**：
   - 修改 `appsettings.Production.json`：
   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=tcp:your-sql-server.database.windows.net,1433;Database=TodoDb;User Id=your-user;Password=your-password;"
     }
   }
   ```

6. **运行数据库迁移**：
   ```bash
   # 在发布目录执行
   dotnet ef database update --environment Production
   ```

**方案 B：Docker 容器化部署（跨平台方案）**

**Dockerfile**（多阶段构建）：
```dockerfile
# 构建阶段
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["TodoApp.Web/TodoApp.Web.csproj", "TodoApp.Web/"]
RUN dotnet restore "TodoApp.Web/TodoApp.Web.csproj"
COPY . .
WORKDIR "/src/TodoApp.Web"
RUN dotnet publish "TodoApp.Web.csproj" -c Release -o /app/publish /p:UseRazorBuildServer=true

# 运行阶段
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 80
ENTRYPOINT ["dotnet", "TodoApp.Web.dll"]
```

**docker-compose.yml**：
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=TodoDb;User Id=sa;Password=YourStrong@Passw0rd;
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong@Passw0rd
    volumes:
      - sqldata:/var/opt/mssql/data
    ports:
      - "1433:1433"

volumes:
  sqldata:
```

**部署命令**：
```bash
# 构建镜像
docker build -t todo-app:latest .

# 使用 Docker Compose 启动（包含应用 + 数据库）
docker-compose up -d

# 查看日志
docker-compose logs -f web

# 执行数据库迁移
docker-compose exec web dotnet ef database update
```

**生产环境建议**：
- 使用 HTTPS（Let's Encrypt 免费证书或购买证书）
- 配置反向代理（Nginx/Apache）处理静态资源和 SSL 终结
- 设置定期备份（SQL Server Agent 或 cron 任务）
- 配置监控（Application Insights / Prometheus）
- 日志集中收集（ELK Stack / Seq）

**涉及文件**：
- `Dockerfile` — Docker 镜像构建配置
- `docker-compose.yml` — 多容器编排
- `.dockerignore` — Docker 忽略文件
- `deploy/iis/` — IIS 部署脚本和配置模板
- `deploy/docker/` — Docker 部署相关文件

---

## 数据库设计

### TodoItem 实体（10 个字段）

```csharp
namespace TodoApp.Web.Models;

public enum Priority
{
    Low = 1,
    Medium = 2,
    High = 3
}

public enum TodoStatus
{
    Pending = 0,      // 待办
    InProgress = 1,   // 进行中
    Completed = 2,    // 已完成
    Cancelled = 3,    // 已取消
    Expired = 4       // 已过期
}

public class TodoItem
{
    public int Id { get; set; }                              // 主键（自增）
    
    [Required]
    [StringLength(200)]
    public string Title { get; set; } = string.Empty;        // 标题（必填，最大200字符）
    
    public string? Description { get; set; }                 // 描述（可选，文本类型）
    
    public Priority Priority { get; set; } = Priority.Medium; // 优先级（默认中等）
    
    public TodoStatus Status { get; set; } = TodoStatus.Pending; // 状态（默认待办）
    
    public DateTime? DueDate { get; set; }                   // 截止日期（可为空）
    
    public bool IsCompleted { get; set; }                    // 是否完成（冗余字段，方便查询）
    
    public string? Tags { get; set; }                        // 标签（JSON 数组格式存储）
    
    public bool IsDeleted { get; set; }                      // 软删除标记
    
    // 时间戳
    public DateTime CreatedAt { get; set; } = DateTime.Now;  // 创建时间
    public DateTime UpdatedAt { get; set; } = DateTime.Now;  // 更新时间
    
    // 外键
    [Required]
    public string UserId { get; set; } = string.Empty;       // 所属用户ID
    
    // 导航属性
    public ApplicationUser User { get; set; }                // 用户导航属性
    
    [Timestamp]
    public byte[] RowVersion { get; set; } = null!;          // 乐观锁版本号
}
```

### ER 图说明（实体关系图）

```
┌─────────────────┐         ┌──────────────────────┐
│  ApplicationUser │         │       TodoItem        │
│─────────────────│         │──────────────────────│
│ PK Id (string)  │───1:N──▶│ PK Id (int)          │
│    UserName     │         │    Title (nvarchar)  │
│    Email        │         │    Description       │
│    PasswordHash │         │    Priority (tinyint)│
│    ...          │         │    Status (tinyint)  │
└─────────────────┘         │    DueDate (datetime)│
                            │    IsCompleted (bit) │
                            │    Tags (nvarchar)   │
                            │    IsDeleted (bit)   │
                            │    CreatedAt         │
                            │    UpdatedAt         │
                            │    UserId (FK)       │◀──N:1
                            │    RowVersion        │
                            └──────────────────────┘

关系说明：
- ApplicationUser (1) --- (N) TodoItem
  一个用户可以拥有多个 Todo 项目
  每个 Todo 必须属于一个用户（UserId 外键非空）
```

### 数据库索引设计

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    
    // TodoItem 索引配置
    modelBuilder.Entity<TodoItem>(entity =>
    {
        entity.ToTable("Todos");
        
        entity.HasIndex(e => e.UserId).HasDatabaseName("IX_Todos_UserId");
        entity.HasIndex(e => e.Status).HasDatabaseName("IX_Todos_Status");
        entity.HasIndex(e => e.Priority).HasDatabaseName("IX_Todos_Priority");
        entity.HasIndex(e => e.DueDate).HasDatabaseName("IX_Todos_DueDate");
        entity.HasIndex(e => e.CreatedAt).HasDatabaseName("IX_Todos_CreatedAt");
        entity.HasIndex(e => new { e.UserId, e.IsDeleted }).HasDatabaseName("IX_Todos_UserId_IsDeleted");
        
        // 复合索引：常用查询条件组合
        entity.HasIndex(e => new { e.UserId, e.Status, e.IsDeleted })
              .HasDatabaseName("IX_Todos_User_Status_Deleted");
              
        entity.HasIndex(e => new { e.UserId, e.CreatedAt, e.IsDeleted })
              .HasDatabaseName("IX_Todos_User_Created_Deleted");
    });
}
```

---

## 4 层架构说明

本项目采用经典的**分层架构（Layered Architecture）**，将应用分为四个职责清晰的层次：

### 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                      │
│                  (Controllers + Views)                       │
│  职责：接收 HTTP 请求、调用 Service、返回 HTML 响应           │
├─────────────────────────────────────────────────────────────┤
│                      Business Layer                         │
│                      (Services)                             │
│  职责：实现业务逻辑、事务协调、数据转换                        │
├─────────────────────────────────────────────────────────────┤
│                    Data Access Layer                         │
│                    (Repositories)                           │
│  职责：封装数据库操作、CRUD 实现、查询构建                     │
├─────────────────────────────────────────────────────────────┤
│                       Data Layer                            │
│              (DbContext + Entity Models)                     │
│  职责：定义数据模型、数据库连接、ORM 映射                      │
└─────────────────────────────────────────────────────────────┘
```

### 第 1 层：Controllers（表现层）

**位置**：`Controllers/`

**职责**：
- 接收和解析 HTTP 请求（路由、查询参数、表单数据）
- 调用 Service 层执行业务逻辑
- 选择 View 并传递 ViewModel
- 处理响应（HTML 重定向、JSON、文件下载等）
- **不应该包含**：业务规则、数据访问代码

**代码示例**：
```csharp
[Authorize]
public class TodoController : Controller
{
    private readonly ITodoService _todoService;
    private readonly IUserService _userService;

    public TodoController(ITodoService todoService, IUserService userService)
    {
        _todoService = todoService;
        _userService = userService;
    }

    // GET: /Todo
    [HttpGet]
    public async Task<IActionResult> Index(
        int page = 1, 
        int pageSize = 10, 
        string? search = null, 
        TodoStatus? status = null)
    {
        var currentUserId = GetCurrentUserId();
        var result = await _todoService.GetPagedAsync(page, pageSize, search, status, currentUserId);
        
        var viewModel = new TodoIndexViewModel
        {
            PagedTodos = result,
            SearchQuery = search,
            SelectedStatus = status,
            Statuses = Enum.GetValues(typeof(TodoStatus)).Cast<TodoStatus>()
        };
        
        return View(viewModel);
    }
}
```

**关键原则**：
- Controller 保持"瘦"（Thin Controller），只做协调工作
- 所有业务逻辑委托给 Service 层
- 使用 ViewModel 而不是直接暴露 Entity

---

### 第 2 层：Services（业务逻辑层）

**位置**：`Services/`

**职责**：
- 实现核心业务规则和流程
- 协调多个 Repository 完成复杂操作
- 事务管理（ACID 保证）
- 数据验证（业务层面）
- DTO/Entity 转换
- 缓存策略（如有）

**接口定义**（`ITodoService.cs`）：
```csharp
public interface ITodoService
{
    Task<PagedResult<TodoDto>> GetPagedAsync(
        int page, int pageSize, string? search, TodoStatus? status, string userId);
    Task<TodoDto?> GetByIdAsync(int id, string userId);
    Task<TodoDto> CreateAsync(CreateTodoRequest request, string userId);
    Task UpdateAsync(int id, UpdateTodoRequest request, string userId);
    Task DeleteAsync(int id, string userId);
    Task ToggleCompleteAsync(int id, string userId);
    Task BatchDeleteAsync(IEnumerable<int> ids, string userId);
    Task<IEnumerable<TagCountDto>> GetTagCountsAsync(string userId);
}
```

**实现示例**（`TodoService.cs`）：
```csharp
public class TodoService : ITodoService
{
    private readonly ITodoRepository _todoRepository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly ILogger<TodoService> _logger;
    private readonly IMapper _mapper;

    public TodoService(
        ITodoRepository todoRepository,
        IUnitOfWork unitOfWork,
        ILogger<TodoService> logger,
        IMapper mapper)
    {
        _todoRepository = todoRepository;
        _unitOfWork = unitOfWork;
        _logger = logger;
        _mapper = mapper;
    }

    public async Task<TodoDto> CreateAsync(CreateTodoRequest request, string userId)
    {
        // 业务规则验证
        if (request.DueDate.HasValue && request.DueDate.Value < DateTime.Today)
        {
            throw new ValidationException("截止日期不能是过去的日期");
        }

        var todo = _mapper.Map<TodoItem>(request);
        todo.UserId = userId;
        todo.CreatedAt = DateTime.UtcNow;
        todo.UpdatedAt = DateTime.UtcNow;

        await _todoRepository.AddAsync(todo);
        await _unitOfWork.SaveChangesAsync();

        _logger.LogInformation("用户 {UserId} 创建了 Todo {TodoId}: {Title}", userId, todo.Id, todo.Title);

        return _mapper.Map<TodoDto>(todo);
    }
}
```

**关键原则**：
- 通过接口编程（面向接口而非实现）
- 使用构造函数注入依赖
- 方法粒度适中（一个方法做一件事）
- 抛出有意义的业务异常

---

### 第 3 层：Repositories（数据访问层）

**位置**：`Repositories/`

**职责**：
- 封装所有数据库 CRUD 操作
- 构建 LINQ 查询（使用 EF Core）
- 实现仓储模式（Repository Pattern）
- 提供泛型基础仓库（BaseRepository）

**接口定义**（`ITodoRepository.cs`）：
```csharp
public interface ITodoRepository : IRepository<TodoItem>
{
    Task<PagedResult<TodoItem>> GetPagedAsync(
        int page, int pageSize,
        Expression<Func<TodoItem, bool>>? filter = null,
        Func<IQueryable<TodoItem>, IOrderedQueryable<TodoItem>>? orderBy = null);
        
    Task<IEnumerable<TodoItem>> GetByUserIdAsync(string userId);
    Task<IEnumerable<TodoItem>> SearchAsync(string keyword, string userId);
    Task<bool> ExistsByTitleAsync(string title, string userId, int? excludeId = null);
}
```

**泛型基础仓库**（`BaseRepository.cs`）：
```csharp
public class BaseRepository<TEntity> : IRepository<TEntity> where TEntity : class
{
    protected readonly ApplicationDbContext _context;
    protected readonly DbSet<TEntity> _dbSet;

    public BaseRepository(ApplicationDbContext context)
    {
        _context = context;
        _dbSet = context.Set<TEntity>();
    }

    public virtual async Task<TEntity?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<TEntity>> GetAllAsync()
    {
        return await _dbSet.AsNoTracking().ToListAsync();
    }

    public virtual async Task AddAsync(TEntity entity)
    {
        await _dbSet.AddAsync(entity);
    }

    public virtual void Update(TEntity entity)
    {
        _dbSet.Update(entity);
    }

    public virtual void Delete(TEntity entity)
    {
        _dbSet.Remove(entity);
    }

    // ... 其他通用方法
}
```

**具体仓库实现**（`TodoRepository.cs`）：
```csharp
public class TodoRepository : BaseRepository<TodoItem>, ITodoRepository
{
    public TodoRepository(ApplicationDbContext context) : base(context) { }

    public async Task<PagedResult<TodoItem>> GetPagedAsync(
        int page, int pageSize,
        Expression<Func<TodoItem, bool>>? filter = null,
        Func<IQueryable<TodoItem>, IOrderedQueryable<TodoItem>>? orderBy = null)
    {
        IQueryable<TodoItem> query = _dbSet
            .Include(t => t.User)
            .AsNoTracking();

        if (filter != null)
            query = query.Where(filter);

        var totalCount = await query.CountAsync();

        if (orderBy != null)
            query = orderBy(query);
        else
            query = query.OrderByDescending(t => t.CreatedAt);

        var items = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToListAsync();

        return new PagedResult<TodoItem>
        {
            Items = items,
            CurrentPage = page,
            PageSize = pageSize,
            TotalCount = totalCount
        };
    }

    public async Task<bool> ExistsByTitleAsync(string title, string userId, int? excludeId = null)
    {
        var query = _dbSet.Where(t => t.Title == title && t.UserId == userId && !t.IsDeleted);
        
        if (excludeId.HasValue)
            query = query.Where(t => t.Id != excludeId.Value);
            
        return await query.AnyAsync();
    }
}
```

**关键原则**：
- 只关注数据访问，不包含业务逻辑
- 使用泛型减少重复代码
- AsNoTracking 用于只读查询提升性能
- Include 解决 N+1 查询问题

---

### 第 4 层：Data（数据层）

**位置**：`Data/`

**职责**：
- 定义 DbContext（数据库上下文）
- 定义 Entity 实体类
- 配置 Fluent API 映射
- 数据库迁移管理
- 种子数据初始化

**ApplicationDbContext.cs**：
```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // DbSet 属性（每个对应一张数据库表）
    public DbSet<TodoItem> Todos { get; set; }
    public DbSet<ApplicationUser> Users { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // 应用所有实体配置（从单独的配置类中加载）
        modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());

        // 全局查询过滤器：软删除
        modelBuilder.Entity<TodoItem>().HasQueryFilter(e => !e.IsDeleted);

        // 种子数据
        SeedData(modelBuilder);
    }

    private static void SeedData(ModelBuilder modelBuilder)
    {
        // 开发环境种子数据（仅在 Development 环境生效）
        // ...
    }
}
```

**实体配置示例**（`TodoItemConfiguration.cs`）：
```csharp
public class TodoItemConfiguration : IEntityTypeConfiguration<TodoItem>
{
    public void Configure(EntityTypeBuilder<TodoItem> builder)
    {
        builder.ToTable("Todos");
        
        builder.HasKey(t => t.Id);
        builder.Property(t => t.Id).UseIdentityColumn();
        
        builder.Property(t => t.Title)
            .IsRequired()
            .HasMaxLength(200)
            .IsUnicode(true);
            
        builder.Property(t => t.Description)
            .HasColumnType("ntext");  // 支持长文本
            
        builder.Property(t => t.Priority)
            .HasConversion<int>();     // 枚举转整数存储
            
        builder.Property(t => t.Status)
            .HasConversion<int>();
            
        builder.Property(t => t.Tags)
            .HasColumnType("nvarchar(max)");  // JSON 格式存储
            
        builder.HasOne(t => t.User)
            .WithMany(u => u.Todos)
            .HasForeignKey(t => t.UserId)
            .OnDelete(DeleteBehavior.Cascade);
            
        builder.Property(t => t.RowVersion)
            .IsRowVersion();  // 乐观锁
    }
}
```

**依赖注入注册**（`Program.cs`）：
```csharp
// 注册 DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        b => b.MigrationsAssembly("TodoApp.Web")));

// 注册 Repository 和 Service
builder.Services.AddScoped<ITodoRepository, TodoRepository>();
builder.Services.AddScoped<ITodoService, TodoService>();

// 注册 UnitOfWork
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

---

## 目录结构

```
todo-app/
├── TodoApp.sln                                # 解决方案文件
│
├── TodoApp.Web/                               # 主 Web 项目（MVC）
│   ├── TodoApp.Web.csproj                     # 项目配置
│   ├── Program.cs                             # 应用程序入口
│   ├── appsettings.json                       # 开发环境配置
│   ├── appsettings.Development.json           # 开发专用配置
│   ├── appsettings.Production.json            # 生产环境配置
│   │
│   ├── Controllers/                           # 控制器层
│   │   ├── HomeController.cs                 # 首页控制器
│   │   ├── AccountController.cs              # 用户认证控制器
│   │   └── TodoController.cs                 # Todo CRUD 控制器
│   │
│   ├── Views/                                 # 视图层
│   │   ├── Home/
│   │   │   ├── Index.cshtml                  # 首页
│   │   │   └── Privacy.cshtml                # 隐私政策页
│   │   ├── Account/
│   │   │   ├── Login.cshtml                  # 登录页
│   │   │   ├── Register.cshtml               # 注册页
│   │   │   └── Logout.cshtml                 # 登出确认页
│   │   ├── Todo/
│   │   │   ├── Index.cshtml                  # Todo 列表页
│   │   │   ├── Details.cshtml                # Todo 详情页
│   │   │   ├── Create.cshtml                 # 创建 Todo 页
│   │   │   ├── Edit.cshtml                   # 编辑 Todo 页
│   │   │   └── Delete.cshtml                 # 删除确认页
│   │   └── Shared/
│   │       ├── _Layout.cshtml               # 主布局
│   │       ├── _LoginPartial.cshtml         # 登录状态部分视图
│   │       ├── _ValidationScriptsPartial.cshtml # 验证脚本引用
│   │       ├── Error.cshtml                 # 错误页
│   │       ├── NotFound.cshtml              # 404 页
│   │       └── Components/
│   │           └── Pagination/             # 分页组件
│   │               └── Default.cshtml      # 分页默认视图
│   │
│   ├── Models/                                # 数据模型
│   │   ├── Entities/                         # 数据库实体
│   │   │   ├── ApplicationUser.cs          # 用户实体
│   │   │   └── TodoItem.cs                 # Todo 实体
│   │   ├── ViewModels/                      # 视图模型
│   │   │   ├── AccountViewModels.cs        # 认证相关 VM
│   │   │   └── TodoViewModels.cs           # Todo 相关 VM
│   │   ├── DTOs/                            # 数据传输对象
│   │   │   ├── TodoDto.cs                 # Todo DTO
│   │   │   └── TodoListDto.cs             # Todo 列表 DTO
│   │   ├── Requests/                        # 请求模型
│   │   │   ├── CreateTodoRequest.cs       # 创建请求
│   │   │   └── UpdateTodoRequest.cs       # 更新请求
│   │   └── Enums/                          # 枚举定义
│   │       ├── Priority.cs                # 优先级枚举
│   │       └── TodoStatus.cs              # 状态枚举
│   │
│   ├── Services/                             # 业务逻辑层（接口 + 实现）
│   │   ├── Interfaces/                      # 服务接口
│   │   │   ├── ITodoService.cs            # Todo 服务接口
│   │   │   ├── IUserService.cs            # 用户服务接口
│   │   │   └── IAuthService.cs            # 认证服务接口
│   │   ├── Implementations/                # 服务实现
│   │   │   ├── TodoService.cs             # Todo 服务实现
│   │   │   ├── UserService.cs             # 用户服务实现
│   │   │   └── AuthService.cs             # 认证服务实现
│   │   └── Profiles/                       # AutoMapper 配置
│   │       └── MappingProfile.cs          # 对象映射配置
│   │
│   ├── Repositories/                        # 数据访问层（接口 + 实现）
│   │   ├── Interfaces/
│   │   │   ├── IRepository.cs             # 泛型仓库接口
│   │   │   ├── ITodoRepository.cs         # Todo 仓库接口
│   │   │   └── IUserRepository.cs         # 用户仓库接口
│   │   ├── Implementations/
│   │   │   ├── BaseRepository.cs          # 泛型基础仓库
│   │   │   ├── TodoRepository.cs          # Todo 仓库实现
│   │   │   └── UserRepository.cs          # 用户仓库实现
│   │   └── Extensions/
│   │       └── QueryableExtensions.cs     # LINQ 扩展方法
│   │
│   ├── Data/                                 # 数据层
│   │   ├── ApplicationDbContext.cs         # 数据库上下文
│   │   ├── Configurations/                  # Fluent API 实体配置
│   │   │   ├── ApplicationUserConfiguration.cs
│   │   │   └── TodoItemConfiguration.cs
│   │   ├── Migrations/                      # 数据库迁移文件
│   │   │   ├── 20260417000001_InitialCreate.cs
│   │   │   ├── 20260417000002_AddTagsToTodo.cs
│   │   │   └── ApplicationDbContextModelSnapshot.cs
│   │   └── SeedData.cs                      # 种子数据初始化
│   │
│   ├── Middleware/                           # 自定义中间件
│   │   └── ExceptionMiddleware.cs          # 全局异常处理中间件
│   │
│   ├── Filters/                              # 过滤器
│   │   └── GlobalExceptionFilter.cs        # 全局异常过滤器
│   │
│   ├── Attributes/                           # 自定义特性
│   │   └── FutureDateValidatorAttribute.cs  # 未来日期验证器
│   │
│   ├── Exceptions/                           # 自定义异常类
│   │   ├── NotFoundException.cs
│   │   ├── ValidationException.cs
│   │   └── BusinessException.cs
│   │
│   ├── Common/                               # 公共工具类
│   │   ├── PagedResult.cs                   # 泛型分页结果
│   │   ├── ApiResponse.cs                  # 统一响应格式
│   │   └── Constants.cs                    # 常量定义
│   │
│   ├── TagHelpers/                           # 自定义 Tag Helper
│   │   └── PaginationTagHelper.cs          # 分页 TagHelper
│   │
│   ├── wwwroot/                              # 静态资源
│   │   ├── css/
│   │   │   ├── bootstrap.min.css
│   │   │   └── site.css                     # 自定义样式
│   │   ├── js/
│   │   │   ├── bootstrap.bundle.min.js
│   │   │   ├── jquery.min.js
│   │   │   ├── jquery.validate.min.js
│   │   │   ├── jquery.validate.unobtrusive.min.js
│   │   │   └── site.js                      # 自定义脚本
│   │   └── images/
│   │
│   └── Properties/
│       └── launchSettings.json              # 启动配置
│
├── TodoApp.Tests/                            # 单元测试项目
│   ├── TodoApp.Tests.csproj
│   ├── Controllers/
│   │   ├── TodoControllerTests.cs
│   │   └── AccountControllerTests.cs
│   ├── Services/
│   │   └── TodoServiceTests.cs
│   ├── Repositories/
│   │   └── TodoRepositoryTests.cs
│   ├── Helpers/
│   │   ├── TestDataGenerator.cs
│   │   └── MockHelpers.cs
│   └── appsettings.json
│
├── deploy/                                   # 部署相关文件
│   ├── iis/
│   │   ├── web.config.template              # IIS 配置模板
│   │   └── deploy.ps1                       # PowerShell 部署脚本
│   └── docker/
│       ├── Dockerfile
│       ├── docker-compose.yml
│       ├── docker-compose.override.yml
│       └── .dockerignore
│
├── .gitignore
├── README.md                                 # 本文件
└── docs/
    ├── architecture.md                       # 架构文档
    ├── database-design.md                    # 数据库设计文档
    └── api-reference.md                     # API 参考文档
```

---

## 运行步骤

### 环境要求

在开始之前，请确保满足以下环境要求：

**必须安装**：
- [ ] .NET 8.0 SDK（运行 `dotnet --version` 确认 ≥ 8.0.100）
- [ ] SQL Server 2019+ 或 SQL Server Developer Edition（推荐 LocalDB 快速启动）
- [ ] Visual Studio 2022（社区版及以上）或 VS Code + C# Dev Kit
- [ ] Git（版本控制）

**推荐安装**（可选但有助于开发）：
- [ ] SQL Server Management Studio (SSMS) — 数据库可视化工具
- [ ] Azure Data Studio — 轻量级数据库管理工具
- [ ] Postman — API 测试工具（如需调试接口）

### 步骤 1：克隆/创建项目（2分钟）

```bash
# 如果是从 GitHub 克隆
git clone https://github.com/your-org/todo-app.git
cd todo-app

# 或者手动创建（参考 hello-world 项目的创建方式）
mkdir todo-app
cd todo-app
dotnet new sln -n TodoApp
# ... （后续步骤会详细说明）
```

### 步骤 2：还原 NuGet 包（1分钟）

```bash
# 还原解决方案的所有 NuGet 依赖
dotnet restore

# 或者分别还原每个项目
dotnet restore TodoApp.Web/TodoApp.Web.csproj
dotnet restore TodoApp.Tests/TodoApp.Tests.csproj
```

**主要依赖包**（自动还原）：
```xml
<!-- TodoApp.Web.csproj 主要依赖 -->
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation" Version="8.0.*" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.*" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="8.0.*" />
  <PackageReference Include="Microsoft.AspNetCore.Authentication.Cookies" Version="2.2.*" />
  <PackageReference Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.*" />
  <PackageReference Include="Serilog.AspNetCore" Version="8.0.*" />
  <PackageReference Include="FluentValidation.AspNetCore" Version="11.3.*" />
  <PackageReference Include="Bootstrap" Version="5.3.*" />
</ItemGroup>

<!-- TodoApp.Tests.csproj 测试依赖 -->
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.*" />
  <PackageReference Include="xunit" Version="2.6.*" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.5.*" />
  <PackageReference Include="Moq" Version="4.20.*" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="8.0.*" />
  <PackageReference Include="coverlet.collector" Version="6.0.*" />
</ItemGroup>
```

### 步骤 3：配置数据库连接（3分钟）

#### 3.1 修改 `appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=TodoAppDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

**连接字符串说明**：
- `(localdb)\\mssqllocaldb`：使用 SQL Server LocalDB（无需安装完整 SQL Server）
- `TodoAppDb`：数据库名称（首次运行时会自动创建）
- `Trusted_Connection=True`：Windows 身份验证（使用当前登录账户）
- `MultipleActiveResultSets=true`：允许单个连接上执行多个查询（MARS 功能）

**如果使用完整版 SQL Server**：
```json
"DefaultConnection": "Server=localhost;Database=TodoAppDb;User Id=sa;Password=YourStrong@Passw0rd;TrustServerCertificate=True;"
```

#### 3.2 验证数据库连接

```bash
# 在项目根目录执行（确保 TodoApp.Web 为启动项目）
cd TodoApp.Web

# 检查是否能连接数据库
dotnet ef database info
```

如果看到类似输出，说明连接成功：
```
Provider name: Microsoft.EntityFrameworkCore.SqlServer
Database name: TodoAppDb
DataSource: (localdb)\mssqllocaldb
...
```

### 步骤 4：执行数据库迁移（2分钟）

#### 4.1 创建初始迁移（如果是全新项目）

```bash
# 在 TodoApp.Web 目录下执行
cd TodoApp.Web

# 创建初始迁移（根据 DbContext 和 Entity 自动生成）
dotnet ef migrations add InitialCreate

# 输出示例：
# Done. To undo this action, use 'ef migrations remove'.
```

#### 4.2 应用迁移到数据库

```bash
# 将迁移应用到数据库（创建表结构）
dotnet ef database update

# 输出示例：
# Applying migration '20260417000001_InitialCreate'.
# Done.
```

**迁移命令速查**：

| 命令 | 用途 |
|------|------|
| `dotnet ef migrations add <名称>` | 创建新的迁移文件 |
| `dotnet ef database update` | 应用待执行的迁移 |
| `dotnet ef database update <名称>` | 回滚到指定迁移 |
| `dotnet ef migrations list` | 查看所有迁移历史 |
| `dotnet ef migrations remove` | 删除最近一次未应用的迁移 |
| `dotnet ef migrations script` | 生成 SQL 脚本（用于生产环境 DBA 审核） |

#### 4.3 验证数据库表结构

打开 SSMS 或 VS Code 的 SQL Server 扩展，连接到数据库后应能看到以下表：

- `__EFMigrationsHistory` — 迁移历史记录表
- `AspNetUsers` — 用户表
- `Todos` — Todo 项主表

### 步骤 5：初始化种子数据（2分钟）

种子数据用于开发和演示，提供预置的用户和 Todo 数据：

**SeedData.cs 内容**：
```csharp
public static class SeedData
{
    public static async Task Initialize(IServiceProvider serviceProvider)
    {
        using var scope = serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();

        // 确保数据库已创建
        context.Database.EnsureCreated();

        // 检查是否已有数据（避免重复插入）
        if (context.Todos.Any())
        {
            return; // 数据库已有数据，跳过种子
        }

        // 创建测试用户
        var testUser = new ApplicationUser
        {
            Id = "test-user-001",
            UserName = "demo@example.com",
            Email = "demo@example.com",
            NormalizedEmail = "DEMO@EXAMPLE.COM",
            EmailConfirmed = true,
            SecurityStamp = Guid.NewGuid().ToString()
        };

        // 注意：实际项目中应该使用 UserManager 创建用户（自动哈希密码）
        context.Users.Add(testUser);

        // 创建示例 Todos
        var todos = new List<TodoItem>
        {
            new()
            {
                Title = "学习 ASP.NET Core 基础",
                Description = "完成官方文档的入门教程，理解 MVC 架构和 Razor 语法。",
                Priority = Priority.High,
                Status = TodoStatus.Completed,
                DueDate = DateTime.Today.AddDays(-5),
                CreatedAt = DateTime.UtcNow.AddDays(-10),
                UserId = testUser.Id,
                Tags = "[\"学习\", \"ASP.NET Core\"]"
            },
            new()
            {
                Title = "搭建开发环境",
                Description = "安装 Visual Studio 2022、SQL Server、Git 等必要工具。",
                Priority = Priority.High,
                Status = TodoStatus.Completed,
                DueDate = DateTime.Today.AddDays(-12),
                CreatedAt = DateTime.UtcNow.AddDays(-15),
                UserId = testUser.Id,
                Tags = "[\"环境配置\"]"
            },
            new()
            {
                Title = "完成 Todo 应用项目",
                Description = "按照教程完成 Todo 应用的全部功能，包括 CRUD、认证、分页、测试等。",
                Priority = Priority.High,
                Status = TodoStatus.InProgress,
                DueDate = DateTime.Today.AddDays(3),
                CreatedAt = DateTime.UtcNow.AddDays(-2),
                UserId = testUser.Id,
                Tags = "[\"项目\", \"实战\"]"
            },
            new()
            {
                Title = "学习 Entity Framework Core",
                Description = "掌握 Code First 迁移、LINQ 查询、关系配置等核心概念。",
                Priority = Priority.Medium,
                Status = TodoStatus.Pending,
                DueDate = DateTime.Today.AddDays(7),
                CreatedAt = DateTime.UtcNow,
                UserId = testUser.Id,
                Tags = "[\"学习\", \"数据库\", \"EF Core\"]"
            },
            new()
            {
                Title = "阅读《Clean Code》",
                Description = "Robert C. Martin 经典著作，提升代码质量和可维护性。",
                Priority = Priority.Low,
                Status = TodoStatus.Pending,
                DueDate = DateTime.Today.AddDays(14),
                CreatedAt = DateTime.UtcNow,
                UserId = testUser.Id,
                Tags = "[\"阅读\", \"书籍\"]"
            },
            new()
            {
                Title = "编写单元测试",
                Description = "为 Todo 服务的核心方法编写单元测试，目标覆盖率 85%+。",
                Priority = Priority.Medium,
                Status = TodoStatus.Pending,
                DueDate = DateTime.Today.AddDays(5),
                CreatedAt = DateTime.UtcNow.AddDays(-1),
                UserId = testUser.Id,
                Tags = "[\"测试\", \"质量保障\"]"
            },
            new()
            {
                Title = "部署到云服务器",
                Description = "将应用部署到 Azure 或阿里云，配置域名和 HTTPS。",
                Priority = Priority.Low,
                Status = TodoStatus.Pending,
                DueDate = DateTime.Today.AddDays(21),
                CreatedAt = DateTime.UtcNow,
                UserId = testUser.Id,
                Tags = "[\"部署\", \"运维\"]"
            },
            new()
            {
                Title = "复习 C# 12 新特性",
                Description = "主构造函数、集合表达式、内联数组等新语法的学习和实践。",
                Priority = Priority.Medium,
                Status = TodoStatus.Expired,
                DueDate = DateTime.Today.AddDays(-1),  // 已过期
                CreatedAt = DateTime.UtcNow.AddDays(-7),
                UserId = testUser.Id,
                Tags = "[\"C#\", \"语言特性\"]"
            }
        };

        context.Todos.AddRange(todos);
        await context.SaveChangesAsync();

        Console.WriteLine($"✅ 种子数据初始化完成！插入了 {todos.Count} 条 Todo 记录。");
    }
}
```

**在 Program.cs 中调用种子数据**：
```csharp
var app = builder.Build();

// 初始化种子数据（仅开发环境）
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    await SeedData.Initialize(scope.ServiceProvider);
}
```

### 步骤 6：运行项目（1分钟）

```bash
# 进入 Web 项目目录
cd TodoApp.Web

# 启动开发服务器（默认 http://localhost:5000）
dotnet run

# 或者使用 Visual Studio
# 按 F5 或点击"启动调试"按钮（IIS Express）
# 或 Ctrl+F5（不调试启动 Kestrel）
```

**预期结果**：
1. 浏览器自动打开首页
2. 点击右上角"注册"创建账户
3. 使用新账户登录
4. 看到 Todo 列表页面（包含 8 条种子数据）
5. 可以尝试创建、编辑、删除、搜索、分页等功能

**默认登录凭据（种子数据）**：
- 邮箱：`demo@example.com`
- 密码：`Demo@123456`（需要在 Identity 中预设）

### 步骤 7：运行单元测试（1分钟）

```bash
# 运行所有测试
dotnet test

# 预期输出：
# 测试运行成功。
# 总计: 9
#     成功: 9
#     总时间: 2.1234 秒
```

---

## 学习要点

通过完成本项目，你将深入掌握以下 **15 个关键知识点**（每个知识点都映射到具体文件和行号）：

### 1. Entity Framework Core Code First 开发流程 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

**核心概念**：先定义 C# 实体类，然后由 EF Core 自动生成数据库表结构。

**关键文件**：
- `Data/ApplicationDbContext.cs:L15-L35` — DbContext 定义
- `Models/Entities/TodoItem.cs:L1-L50` — TodoItem 实体定义
- `Data/Configurations/TodoItemConfiguration.cs:L1-L45` — Fluent API 配置
- `Data/Migrations/20260417000001_InitialCreate.cs` — 迁移文件（自动生成）

**学习要点**：
- DbContext 作为数据库会话的入口点
- DbSet\<T\> 属性对应数据库表
- OnModelCreating 方法配置实体映射
- Key/Required/MaxLength/Table 等配置 API
- Migration 工作流：Add → Update → Remove

**关联教程**：[[基础篇/02-EF-Core入门]]

---

### 2. 依赖注入（DI）与 IoC 容器 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

**核心概念**：通过构造函数注入依赖，而不是在类内部创建实例（控制反转）。

**关键文件**：
- `Program.cs:L25-L55` — DI 容器注册
- `Services/Interfaces/ITodoService.cs:L1-L15` — 服务接口定义
- `Services/Implementations/TodoService.cs:L1-L30` — 服务实现（构造函数注入）
- `Controllers/TodoController.cs:L15-L25` — Controller 注入 Service

**DI 生命周期对比**：

| 生命周期 | 注册方式 | 适用场景 | 实例数量 |
|---------|---------|---------|---------|
| Transient | `AddTransient<T>()` | 无状态、轻量级服务 | 每次请求新建 |
| Scoped | `AddScoped<T>()` | 有状态服务（如 DbContext、Repository） | 每个 HTTP 请求一个 |
| Singleton | `AddSingleton<T>()` | 全局配置、缓存服务 | 整个应用只有一个 |

**本项目实践**：
- DbContext、Repository、Service 都注册为 `Scoped`（每个请求共享同一实例）
- Logger、Mapper 等无状态服务注册为 `Transient`
- Configuration、Cache 等全局资源注册为 `Singleton`

**关联教程**：[[基础篇/01-依赖注入详解]]

---

### 3. Repository 模式与数据访问抽象 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

**核心概念**：将数据访问逻辑封装在独立的 Repository 类中，使上层不直接依赖 EF Core。

**关键文件**：
- `Repositories/Interfaces/IRepository.cs:L1-L30` — 泛型仓库接口
- `Repositories/Implementations/BaseRepository.cs:L1-L80` — 泛型基础仓库实现
- `Repositories/Interfaces/ITodoRepository.cs:L1-L25` — Todo 专用仓库接口
- `Repositories/Implementations/TodoRepository.cs:L1-L90` — Todo 仓库实现

**优势**：
- **可测试性**：可以用 Mock 替换真实数据库
- **可替换性**：未来可切换到其他 ORM（如 Dapper、NHibernate）
- **单一职责**：数据访问逻辑集中在 Repository 中
- **复用性**：BaseRepository 减少重复代码

**关联教程**：[[基础篇/03-Repository模式]]

---

### 4. AutoMapper 对象映射 ⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

**核心概念**：Entity ↔ DTO/ViewModel 之间的自动属性映射。

**关键文件**：
- `Services/Profiles/MappingProfile.cs:L1-L40` — 映射配置
- `Services/Implementations/TodoService.cs:L45-L55` — 使用 Mapper
- `DTOs/TodoDto.cs:L1-L20` — DTO 定义
- `Models/ViewModels/TodoViewModels.cs:L1-L30` — ViewModel 定义

**映射示例**：
```csharp
// MappingProfile.cs
public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<TodoItem, TodoDto>()
            .ForMember(dest => dest.StatusName, opt => opt.MapFrom(src => src.Status.ToString()))
            .ForMember(dest => dest.PriorityName, opt => opt.MapFrom(src => src.Priority.ToString()))
            .ForMember(dest => dest.TagsList, opt => opt.MapFrom(src => JsonSerializer.Deserialize<List<string>>(src.Tags ?? "[]")));
        
        CreateMap<CreateTodoRequest, TodoItem>();
        CreateMap<UpdateTodoRequest, TodoItem>();
    }
}
```

**为什么要用 DTO 而不是直接返回 Entity？**
- 安全性：隐藏敏感字段（如 PasswordHash、RowVersion）
- 性能：Select 投影只查询需要的字段
- 解耦：API 契约与数据库结构独立演进
- 验证：可以在 DTO 上添加不同的验证规则

**关联教程**：[[进阶篇/02-DTO与AutoMapper]]

---

### 5. 表单验证体系（三层验证）⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

**关键文件**：
- `Models/ViewModels/TodoViewModels.cs:L15-L45` — Data Annotations
- `Attributes/FutureDateValidatorAttribute.cs:L1-L25` — 自定义验证器
- `Controllers/TodoController.cs:L65-L95` — ModelState 验证
- `Views/Todo/Create.cshtml:L10-L30` — 客户端验证 Tag Helpers

**三层验证协作流程**：
```
用户输入
    ↓
[客户端 JavaScript 验证] ← jQuery Validation（即时反馈）
    ↓
[Model Binding] ← 自动将表单数据绑定到 ViewModel
    ↓
[Data Annotations] ← Required/StringLength/Range 等
    ↓
[自定义验证器] ← FutureDateValidator 等
    ↓
[Remote 验证] ← AJAX 异步校验（唯一性检查等）
    ↓
[Controller ModelState.IsValid] ← 最终验证汇总
    ↓
通过 → 执行业务逻辑
失败 → 返回视图 + 错误消息
```

**关联教程**：[[基础篇/04-表单处理与验证]]

---

### 6. 分页查询实现 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

**关键文件**：
- `Common/PagedResult.cs:L1-L25` — 泛型分页结果类
- `Repositories/Implementations/TodoRepository.cs:L35-L70` — 分页查询实现
- `Services/Implementations/TodoService.cs:L28-L48` — Service 层封装
- `TagHelpers/PaginationTagHelper.cs:L1-L60` — 分页 UI 组件
- `Views/Todo/Index.cshtml:L80-L95` — 分页组件调用

**分页算法核心**：
```csharp
// Skip = (当前页 - 1) × 每页数量
// Take = 每页数量
// TotalPages = Math.Ceiling(总记录数 / 每页数量)

var items = await query
    .Skip((page - 1) * pageSize)  // 跳过前面的记录
    .Take(pageSize)                // 取当前页的记录
    .ToListAsync();
```

**关联教程**：[[进阶篇/05-分页与排序]]

---

### 7. Cookie 认证机制 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

**关键文件**：
- `Program.cs:L60-L75` — 认证中间件配置
- `Services/Implementations/AuthService.cs:L1-L80` — 登录/注册逻辑
- `Controllers/AccountController.cs:L25-L90` — 认证控制器
- `Views/Account/Login.cshtml:L1-L40` — 登录表单

**认证流程**：
```
用户提交用户名/密码
    ↓
AuthService.ValidateCredentials() → 验证凭据
    ↓
ClaimsPrincipal creation → 创建用户身份标识
    ↓
HttpContext.SignInAsync() → 写入 Authentication Cookie
    ↓
后续请求携带 Cookie → Cookie Authentication Middleware 验证
    ↓
[Authorize] 特性检查 → 允许/拒绝访问
```

**关联教程**：[[基础篇/06-Cookie认证]]

---

### 8. 全局异常处理 ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

**关键文件**：
- `Middleware/ExceptionMiddleware.cs:L1-L80` — 异常中间件
- `Filters/GlobalExceptionFilter.cs:L1-L40` — 全局过滤器
- `Exceptions/NotFoundException.cs:L1-L15` — 自定义异常
- `Program.cs:L120-L125` — 中间件注册顺序

**异常处理层级**：
1. **Action 内 try-catch**：精确处理已知异常
2. **Controller Filter**：统一处理 Controller 层异常
3. **全局 Middleware**：兜底处理所有未捕获异常
4. **DeveloperExceptionPage**：开发环境详细错误页
5. **Production Exception Handler**：生产环境友好错误页

**关联教程**：[[进阶篇/01-全局异常处理]]

---

### 9. LINQ 查询与表达式树 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

**关键文件**：
- `Repositories/Implementations/TodoRepository.cs:L50-L90` — 动态查询构建
- `Services/Implementations/TodoService.cs:L55-L85` — 搜索过滤逻辑
- `Repositories/Extensions/QueryableExtensions.cs:L1-L40` — LINQ 扩展方法

**常用 LINQ 操作符**：
```csharp
// Where：过滤
.Where(t => t.Status == TodoStatus.Pending && t.UserId == userId)

// OrderBy/OrderByDescending：排序
.OrderByDescending(t => t.CreatedAt)

// Select：投影（DTO 转换）
.Select(t => new TodoListDto { Title = t.Title, Status = t.Status })

// Include：预加载关联实体
.Include(t => t.User)

// AsNoTracking：只读查询优化
.AsNoTracking()

// Skip/Take：分页
.Skip((page - 1) * pageSize).Take(pageSize)

// Any/All/Contains：聚合判断
.Any(t => t.Title.Contains(keyword))

// GroupBy：分组统计
.GroupBy(t => t.Status).Select(g => new { Status = g.Key, Count = g.Count() })
```

**动态查询构建**（搜索过滤场景）：
```csharp
IQueryable<TodoItem> query = _context.Todos;

if (!string.IsNullOrWhiteSpace(searchKeyword))
    query = query.Where(t => t.Title.Contains(searchKeyword) || t.Description!.Contains(searchKeyword));

if (status.HasValue)
    query = query.Where(t => t.Status == status.Value);

if (priority.HasValue)
    query = query.Where(t => t.Priority == priority.Value);
```

**关联教程**：[[基础篇/02-EF-Core入门#LINQ查询]]

---

### 10. UnitOfWork 工作单元模式 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

**核心概念**：将多个 Repository 操作包裹在一个数据库事务中，保证原子性。

**关键文件**：
- `Data/IUnitOfWork.cs:L1-L20` — UnitOfWork 接口
- `Data/UnitOfWork.cs:L1-L50` — UnitOfWork 实现
- `Services/Implementations/TodoService.cs:L90-L110` — 事务使用示例

**使用场景**：
```csharp
public async Task TransferTodoOwnershipAsync(int todoId, string newOwnerId, string currentOwnerId)
{
    using var transaction = await _unitOfWork.BeginTransactionAsync();
    try
    {
        var todo = await _todoRepository.GetByIdAsync(todoId);
        if (todo == null || todo.UserId != currentOwnerId)
            throw new NotFoundException("Todo 不存在或无权操作");

        todo.UserId = newOwnerId;
        todo.UpdatedAt = DateTime.UtcNow;
        _todoRepository.Update(todo);

        // 记录转移日志
        var log = new TransferLog { TodoId = todoId, FromUserId = currentOwnerId, ToUserId = newOwnerId };
        await _logRepository.AddAsync(log);

        await _unitOfWork.SaveChangesAsync(); // 提交事务
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync(); // 回滚事务
        throw;
    }
}
```

**关联教程**：[[进阶篇/03-事务管理]]

---

### 11. 异步编程 async/await ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐

**核心概念**：所有 I/O 操作（数据库、网络、文件）都应该使用异步方法。

**关键文件**：整个项目的 Controller、Service、Repository 层

**最佳实践**：
```csharp
// ✅ 正确：全程异步
public async Task<IActionResult> Index()
{
    var todos = await _todoService.GetAllAsync();  // 异步等待
    return View(todos);  // 同步返回 ViewResult
}

// ❌ 错误：混合使用同步和异步（可能导致死锁）
public IActionResult Index()
{
    var todos = _todoService.GetAllAsync().Result;  // 同步阻塞等待（危险！）
    return View(todos);
}

// ✅ 正确：取消令牌支持（长耗时操作）
public async Task<IActionResult> Export(CancellationToken cancellationToken)
{
    var data = await _todoService.ExportAsync(cancellationToken);
    return File(data, "text/csv", "todos.csv");
}
```

**何时使用 async？**
- 数据库操作：`ToListAsync()`, `SaveChangesAsync()`, `FindAsync()`
- 文件操作：`ReadAllTextAsync()`, `WriteAllAsync()`
- 网络请求：`HttpClient.GetAsync()`, `SendAsync()`
- **不要**对纯 CPU 计算使用 async（如 LINQ to Objects 的 Where/Select）

**关联教程**：[[基础篇/07-异步编程详解]]

---

### 12. 日志记录（Logging）⭐⭐
**重要性**：★★★★☆ | **难度**：⭐

**关键文件**：
- `Program.cs:L10-L15` — 日志提供者配置
- `Services/Implementations/TodoService.cs:L12,L45,L78` — 日志调用点
- `Controllers/TodoController.cs:L18,L67,L102` — Controller 日志

**日志级别使用规范**：

| 级别 | 用途 | 示例 |
|------|------|------|
| Trace | 最详细的调试信息（通常禁用） | 方法入参、出参 |
| Debug | 调试期间的诊断信息 | 查询语句、变量值 |
| Information | 正常业务流程的关键节点 | 用户登录、创建 Todo |
| Warning | 异常但可恢复的情况 | 请求参数缺失、使用默认值 |
| Error | 需要关注的错误 | 数据库连接失败、外部 API 超时 |
| Critical | 严重故障 | 应用程序崩溃、数据丢失风险 |

**结构化日志示例**：
```csharp
_logger.LogInformation(
    "用户 {UserId} 创建了 Todo {TodoId}，标题：{Title}",
    userId, todo.Id, todo.Title);
// 输出：用户 user-001 创建了 Todo 42，标题：学习 EF Core
```

**关联教程**：[[进阶篇/04-日志与监控]]

---

### 13. 配置管理（Options Pattern）⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐

**关键文件**：
- `appsettings.json:L15-L20` — 连接字符串配置
- `appsettings.Development.json` — 开发环境覆盖
- `appsettings.Production.json` — 生产环境配置
- `Program.cs:L20-L23` — Options 注册

**多环境配置优先级**：
```
appsettings.json (基础)
    ↓ 被 appsettings.{Environment}.json 覆盖
    ↓ 被环境变量覆盖
    ↓ 被命令行参数覆盖（最高优先级）
```

**强类型配置绑定**：
```csharp
// 定义配置类
public class DatabaseSettings
{
    public string ConnectionString { get; set; } = string.Empty;
    public int CommandTimeout { get; set; } = 30;
    public int MaxRetryCount { get; set; } = 3;
}

// 注册
builder.Services.Configure<DatabaseSettings>(
    builder.Configuration.GetSection("Database"));

// 使用（通过 IOptions<T> 注入）
public class TodoService
{
    private readonly DatabaseSettings _dbSettings;
    
    public TodoService(IOptions<DatabaseSettings> dbOptions)
    {
        _dbSettings = dbOptions.Value;
    }
}
```

**关联教程**：[[基础篇/08-配置管理]]

---

### 14. 单元测试与 Mock ⭐⭐⭐
**重要性**：★★★★★ | **难度**：⭐⭐⭐

**关键文件**：
- `TodoApp.Tests/Controllers/TodoControllerTests.cs:L1-L150` — Controller 测试
- `TodoApp.Tests/Services/TodoServiceTests.cs:L1-L120` — Service 测试
- `TodoApp.Tests/Helpers/MockHelpers.cs:L1-L50` — Mock 工具函数
- `TodoApp.Tests/TodoApp.Tests.csproj:L15-L30` — 测试依赖

**AAA 模式（Arrange-Act-Assert）**：
```csharp
[Fact]
public async Task Create_ValidInput_ReturnsCreatedResult()
{
    // Arrange（准备）：设置 Mock 数据和预期
    var request = new CreateTodoRequest { Title = "Test Todo" };
    var expectedDto = new TodoDto { Id = 1, Title = "Test Todo" };
    _mockTodoService
        .Setup(s => s.CreateAsync(It.IsAny<CreateTodoRequest>(), It.IsAny<string>()))
        .ReturnsAsync(expectedDto);

    // Act（执行）：调用被测方法
    var result = await _controller.Create(request);

    // Assert（断言）：验证结果符合预期
    var redirectResult = Assert.IsType<RedirectToActionResult>(result);
    Assert.Equal("Index", redirectResult.ActionName);
    _mockTodoService.Verify(s => s.CreateAsync(request, It.IsAny<string>()), Times.Once);
}
```

**Mock 最佳实践**：
- Mock 接口，不 Mock 具体类（除非使用 Moq 的 Proxy 功能）
- 只 Mock 需要的方法，不要 Setup 所有方法
- 使用 `It.IsAny<T>()` 匹配任意参数
- 使用 `It.Is<T>(predicate)` 匹配特定条件的参数
- 验证调用次数：`Times.Once()`, `Times.Never()`, `Times.AtLeast(1)`

**关联教程**：[[进阶篇/06-单元测试最佳实践]]

---

### 15. 性能优化技巧 ⭐⭐⭐
**重要性**：★★★★☆ | **难度**：⭐⭐⭐

**关键文件**：
- `Repositories/Implementations/BaseRepository.cs:L35-L45` — AsNoTracking
- `Services/Implementations/TodoService.cs:L62-L72` — Select 投影
- `Program.cs:L90-L105` — 响应压缩配置
- `Program.cs:L55-L60` — 静态文件缓存

**优化技术总结**：

| 优化技术 | 适用场景 | 性能提升 | 本项目应用位置 |
|---------|---------|---------|--------------|
| AsNoTracking | 只读查询 | 30-50% | Repository 基类 |
| Select 投影 | 减少传输字段 | 60-80% | Service 层 DTO 转换 |
| Include 预加载 | 避免 N+1 查询 | 显著 | 详情查询 |
| 响应压缩 | 文本资源 | 60-80% | 中间件管道 |
| 异步编程 | I/O 操作 | 吞吐量提升 | 全项目 |
| 连接池 | 数据库连接 | 减少连接开销 | EF Core 默认 |
| 索引优化 | 大数据量查询 | 10-100x | Fluent API 配置 |
| 分页 | 大列表展示 | 内存降低 | PagedResult |

**关联教程**：[[高手篇/01-性能优化指南]]

---

## 扩展练习

完成基础项目后，尝试以下 **5 个进阶任务** 来进一步提升技能：

### 🎯 练习 1：添加标签分类系统（难度：⭐⭐⭐）
**目标**：实现 Todo 的多标签分类功能，支持标签的 CRUD 管理。

**要求**：
- 新建 `Tag` 实体和 `TagRepository`
- Todo 与 Tag 多对多关系（中间表 `TodoTags`）
- 标签管理页面（创建/编辑/删除标签）
- 创建/编辑 Todo 时可选择多个标签（Checkbox 或 Tag Input 组件）
- 标签云展示（按使用频率调整字体大小）
- 按标签过滤 Todo

**涉及知识点**：EF Core 多对多关系、复杂表单、标签输入组件

**预估工作量**：3-4 小时

---

### 🎯 练习 2：实现优先级与提醒通知（难度：⭐⭐⭐）
**目标**：为 Todo 添加优先级管理和到期提醒功能。

**要求**：
- 优先级可视化（颜色编码：红色-高/黄色-中/绿色-低）
- 到期前 24 小时邮件提醒（Hangfire 后台任务）
- 过期 Todo 自动标红并显示"已过期"徽章
- 优先级排序拖拽调整（前端 jQuery UI Sortable）
- 今日到期 Todo 首页醒目提示

**涉及知识点**：Hangfire 定时任务、邮件发送（MailKit）、前端拖拽排序

**预估工作量**：4-5 小时

---

### 🎯 练习 3：实现拖拽看板视图（难度：⭐⭐⭐⭐）
**目标**：将 Todo 列表改为 Kanban 看板形式（类似 Trello）。

**要求**：
- 三列看板：待办 | 进行中 | 已完成
- 卡片支持拖拽在不同列之间移动（HTML5 Drag and Drop API）
- 拖拽后自动更新 Todo 状态（AJAX 请求）
- 看板数据本地缓存（减少服务器请求）
- 响应式布局（移动端横向滚动）

**涉及知识点**：HTML5 Drag & Drop API、AJAX 无刷新更新、CSS Grid/Flexbox 布局

**预估工作量**：5-6 小时

---

### 🎯 练习 4：数据导出功能（难度：⭐⭐⭐）
**目标**：支持将 Todo 数据导出为多种格式。

**要求**：
- CSV 导出（使用 CsvHelper 库）
- Excel 导出（使用 ClosedXML 或 EPPlus）
- PDF 报告导出（使用 QuestPDF 或 iTextSharp）
- 支持导出筛选后的数据（保留当前搜索条件）
- 导出进度条（大数据量异步导出）
- 导出历史记录（记录每次导出的时间和范围）

**涉及知识点**：文件下载、第三方库集成、异步任务处理

**预估工作量**：3-4 小时

---

### 🎯 练习 5：实时协作功能（难度：⭐⭐⭐⭐⭐）
**目标**：多人同时查看和编辑同一个 Todo 列表，实时同步变化。

**要求**：
- SignalR 实时推送（新增/修改/删除 Todo 时广播给所有在线用户）
- 在线用户列表显示
- 冲突解决（乐观并发，最后写入胜出或合并策略）
- 编辑锁定（某人正在编辑某条 Todo 时提示其他人）
- 操作日志（谁在什么时间做了什么修改）

**涉及知识点**：SignalR 实时通信、并发控制、WebSocket

**预估工作量**：8-10 小时

---

## 常见问题 FAQ

### Q1：数据库迁移失败怎么办？
**A**：常见原因及解决方案：
- **连接字符串错误**：检查 `appsettings.json` 中的 ConnectionStrings 是否正确
- **实体类修改冲突**：如果手动改了数据库但没同步迁移文件，执行 `dotnet ef migrations remove` 删除最后一次迁移，然后重新生成
- **数据丢失风险**：生产环境务必先备份数据库，再执行 `dotnet ef database update`
- **SQL Server 版本不兼容**：确保使用的 EF Core 版本与 SQL Server 版本匹配

### Q2：如何切换到 MySQL 或 PostgreSQL？
**A**：
1. 安装对应的 NuGet 包：
   - MySQL：`Pomelo.EntityFrameworkCore.MySql`
   - PostgreSQL：`Npgsql.EntityFrameworkCore.PostgreSQL`
2. 修改 `Program.cs` 中的数据库提供者：
   ```csharp
   // MySQL
   options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));
   
   // PostgreSQL
   options.UseNpgsql(connectionString);
   ```
3. 更新连接字符串格式（各数据库略有不同）
4. 重新生成迁移：`dotnet ef migrations add InitialCreate --force`（会覆盖原有迁移）

### Q3：单元测试报错"无法解析服务"？
**A**：这是因为在测试环境中没有配置 DI 容器。解决方案：
- 使用 Mock 框架（Moq）模拟所有依赖接口
- 不要在测试中直接实例化需要 DI 的类
- 参考 `TodoApp.Tests/Helpers/MockHelpers.cs` 中的辅助方法

### Q4：如何添加新的 Todo 字段？
**A**：
1. 在 `TodoItem` 实体类中添加新属性
2. 如果需要特殊配置，在 `TodoItemConfiguration.cs` 中添加 Fluent API 配置
3. 执行 `dotnet ef migrations add AddNewField` 生成迁移
4. 执行 `dotnet ef database update` 应用迁移
5. 更新相关的 ViewModel、DTO、Create/Edit 视图

### Q5：部署到生产环境后 500 错误？
**A**：排查步骤：
1. 检查 `appsettings.Production.json` 配置是否正确
2. 查看服务器上的日志文件（默认在 `logs/` 目录）
3. 确认数据库连接字符串有效（测试数据库可达性）
4. 确认所有迁移都已执行：`dotnet ef database update`
5. 检查 IIS/Docker 的错误日志
6. 临时设置 `ASPNETCORE_ENVIRONMENT=Development` 以获取详细错误信息（仅排查时使用）

---

## 关联教程索引

| 知识点 | 教程位置 | 学习优先级 | 本项目应用位置 |
|--------|----------|-----------|---------------|
| 依赖注入 | [[基础篇/01-依赖注入详解]] | 必学 | Program.cs, Service/Controller |
| EF Core 入门 | [[基础篇/02-EF-Core入门]] | 必学 | Data/, Migrations/ |
| Repository 模式 | [[基础篇/03-Repository模式]] | 推荐 | Repositories/ |
| 表单验证 | [[基础篇/04-表单处理与验证]] | 必学 | Models/ViewModels/, Attributes/ |
| 路由系统 | [[基础篇/05-路由系统详解]] | 推荐 | Controllers/, Program.cs |
| Cookie 认证 | [[基础篇/06-Cookie认证]] | 必学 | AccountController, AuthService |
| 异步编程 | [[基础篇/07-异步编程详解]] | 推荐 | 全项目 async/await |
| 配置管理 | [[基础篇/08-配置管理]] | 推荐 | appsettings.*, Options Pattern |
| 全局异常处理 | [[进阶篇/01-全局异常处理]] | 进阶必学 | Middleware/, Filters/ |
| DTO 与 AutoMapper | [[进阶篇/02-DTO与AutoMapper]] | 进阶必学 | Services/Profiles/, DTOs/ |
| 事务管理 | [[进阶篇/03-事务管理]] | 进阶推荐 | UnitOfWork, Service |
| 日志与监控 | [[进阶篇/04-日志与监控]] | 进阶推荐 | ILogger 使用 |
| 分页与排序 | [[进阶篇/05-分页与排序]] | 进阶必学 | PagedResult, PaginationTagHelper |
| 单元测试 | [[进阶篇/06-单元测试最佳实践]] | 进阶必学 | TodoApp.Tests/ |
| 性能优化 | [[高手篇/01-性能优化指南]] | 高级选学 | AsNoTracking, Select, Compression |
| Docker 部署 | [[高手篇/03-Docker容器化部署]] | 高级选学 | Dockerfile, docker-compose.yml |

---

## 下一步学习路线

完成本项目后，你已经具备了构建中型 Web 应用的能力。接下来建议：

### 📚 推荐学习顺序

1. **[[blog-system/README.md]]** — 博客系统（微服务架构预备版）
   - 学习 RESTful API 设计、JWT 认证、富文本编辑、全文搜索
   - 从 MVC 转向 Web API 开发模式
   - 预计耗时：15-20 小时
   - 难度：⭐⭐⭐⭐ 高级

2. **[[ecommerce-mall/README.md]]** — CloudMall 微服务电商商城
   - 学习微服务架构、分布式事务、消息队列、容器编排
   - 掌握企业级系统的设计与实现
   - 预计耗时：30-40 小时
   - 难度：⭐⭐⭐⭐⭐ 专家级

---

## 项目统计

- **总代码量**：~4500 行（C# + Razor + CSS/JS + 测试）
- **文件数量**：80+ 个文件
- **数据库表**：2 张主表（Users + Todos）
- **API 端点**：~15 个（Account + Todo）
- **测试用例**：9 个核心测试（可扩展至 30+）
- **涉及技术点**：15 个核心知识点
- **预计完成时间**：5-8 小时（有 C# 基础）
- **前置知识要求**：完成 Hello World 项目、熟悉 C# 基础语法、了解 HTML/CSS 基础

---

## 许可证

本项目仅用于学习和教育目的。代码基于 MIT 许可证开源。

---

**最后更新时间**：2026-04-17
**维护者**：ASP.NET Core 知识库团队
**反馈渠道**：如有问题或建议，欢迎提交 Issue 或 Pull Request
