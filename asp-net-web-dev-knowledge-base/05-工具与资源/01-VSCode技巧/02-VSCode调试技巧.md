# VS Code 调试技巧完全指南

> 调试是软件开发中最重要的技能之一。本文将深入讲解 VS Code 的调试功能，从基础操作到高级断点技术，从 launch.json 配置到 ASP.NET Core 专项调试场景，帮助你成为调试专家。

---

## 目录

- [一、调试基础](#一调试基础)
- [二、高级断点技术](#二高级断点技术)
- [三、调试窗口详解](#三调试窗口详解)
- [四、launch.json 配置详解](#四launchjson-配置详解)
- [五、ASP.NET Core 常见调试场景](#五aspnet-core-常见调试场景)
- [六、调试最佳实践](#六调试最佳实践)

---

## 一、调试基础

### 核心调试快捷键

VS Code 的调试操作围绕五个核心按键展开，这是每个开发者必须熟练掌握的：

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `F5` | 开始/继续调试 | 启动调试会话或继续执行到下一个断点 |
| `F9` | 切换断点 | 在当前行设置或取消断点 |
| `F10` | 单步跳过 (Step Over) | 执行当前行，不进入函数内部 |
| `F11` | 单步进入 (Step Into) | 进入当前行的函数内部执行 |
| `Shift+F11` | 单步跳出 (Step Out) | 执行完当前函数剩余代码并返回调用者 |

### 启动调试的几种方式

**方式1：直接按 F5**

最简单的方式。VS Code 会自动检测项目类型并使用默认配置启动。

**方式2：选择启动配置**

点击运行视图中的"运行和调试"按钮（或按 `Ctrl+Shift+D`），选择或创建 launch 配置：

```
┌─────────────────────────────────────┐
│  RUN AND DEBUG                      │
│                                     │
│  ▶ .NET Core Launch (web)          │
│    - 使用 IIS Express 启动 Web 应用 │
│                                     │
│  ▶ .NET Core Launch (console)      │
│    - 运行控制台应用程序              │
│                                     │
│  ▶ .NET Core Attach to Process     │
│    - 附加到正在运行的进程            │
│                                     │
│  + 创建 launch.json 文件            │
└─────────────────────────────────────┘
```

**方式3：使用命令面板**

`Ctrl+Shift+P` 输入 "Debug: Start Debugging"

### 第一次调试 ASP.NET Core 项目

当你第一次在 VS Code 中打开 ASP.NET Core 项目时，C# Dev Kit 会提示你生成必要的配置文件：

```bash
# 需要生成的文件：
.vscode/
├── launch.json      # 调试启动配置
├── tasks.json       # 构建任务配置
└── settings.json    # 工作区设置（可选）
```

### 基本调试流程

```
1. 打开包含 .csproj 或 .sln 的项目文件夹

2. 确保已安装 C# Dev Kit 扩展

3. 在需要检查的代码行按 F9 设置断点
   （行号左侧会出现红色圆点）

4. 按 F5 启动调试

5. 当程序执行到断点时会自动暂停

6. 使用 F10/F11 逐步执行代码

7. 查看变量窗口中的值变化

8. 按 Shift+F5 停止调试
```

---

## 二、高级断点技术

普通断点只是调试的基础，VS Code 提供了多种高级断点类型，可以让你更精准地控制调试过程。

### 2.1 条件断点 (Conditional Breakpoints)

条件断点只在满足特定条件时才会暂停程序执行，这对于循环中特定迭代或复杂逻辑的调试非常有用。

#### 设置方法

**方法1：右键菜单**
1. 右键点击断点（红色圆点）
2. 选择"编辑断点"
3. 输入条件表达式

**方法2：内联编辑**
1. 将鼠标悬停在断点上
2. 点击出现的编辑图标（铅笔）

#### 条件类型

##### 表达式条件

当表达式的值为 true 时触发断点：

```csharp
// 场景：调试用户列表处理，只在第100个用户处暂停
foreach (var user in users)
{
    // 断点条件: user.Id == 100
    var result = ProcessUser(user);
}

// 场景：调试错误状态的处理
var response = await _httpClient.GetAsync(url);
// 断点条件: !response.IsSuccessStatusCode
var content = await response.Content.ReadAsStringAsync();
```

##### 命中次数条件 (Hit Count)

根据断点被触发的次数来决定是否暂停：

```
命中次数选项：
├── 等于 (equals)        # 第 N 次命中时暂停
├── 大于等于 (>=)        # 至少第 N 次命中后暂停
├── 是倍数 (multiple of) # 每 N 次命中暂停一次
└── 总是 (always)        # 每次都暂停（默认行为）
```

实际应用示例：

```csharp
// 场景1：只关心第1000次请求
// 命中条件：等于 1000

// 场景2：系统稳定运行一段时间后再观察
// 命中条件：大于等于 5000

// 场景3：每100次请求采样一次
// 命中条件：是 100 的倍数
for (int i = 0; i < 10000; i++)
{
    ProcessRequest(i); // 在此设断点
}
```

### 2.2 日志断点 (Logpoints)

日志断点是 VS Code 最实用的功能之一。它不会暂停程序执行，而是在控制台输出自定义信息，非常适合生产环境调试或不想中断流程的场景。

#### 设置方法

1. 右键点击行号 -> 选择"添加日志点"
2. 或者右键现有断点 -> 编辑断点 -> 切换为日志点模式
3. 输入要输出的消息（支持 `{expression}` 语法引用变量）

#### 实际应用示例

```csharp
// API 控制器中使用日志断点
[HttpGet("{id}")]
public async Task<ActionResult<User>> GetUser(int id)
{
    // 日志断点消息:
    // "API请求: GetUser, id={id}, 时间={DateTime.Now}"
    
    var user = await _userService.GetByIdAsync(id);
    
    // 日志断点消息:
    // "查询结果: found={user != null}, userId={user?.Id}"
    
    return user == null ? NotFound() : Ok(user);
}
```

输出效果：

```
API请求: GetUser, id=42, 时间=2026/4/17 10:30:15
查询结果: found=True, userId=42
API请求: GetUser, id=99999, 时间=2026/4/17 10:30:16
查询结果: found=False, userId=
```

#### 高级日志断点技巧

```csharp
// 1. 格式化复杂对象
// 日志消息: "User对象: {JsonConvert.SerializeObject(user)}"

// 2. 记录性能指标
var stopwatch = Stopwatch.StartNew();
// ... 一些操作 ...
stopwatch.Stop();
// 日志消息: "操作耗时(ms): {stopwatch.ElapsedMilliseconds}"

// 3. 条件性日志（结合表达式）
// 日志消息（仅在错误时输出）:
// "⚠ 错误响应: status={response.StatusCode} for {url}"
// 注意：虽然不能直接加 if，但可以用三元表达式：
// "{(response.IsSuccessStatusCode ? \"OK\" : $\"ERROR:{response.StatusCode}\")}"
```

### 2.3 函数断点 (Function Breakpoints)

函数断点允许你在不打开源文件的情况下，直接通过函数名设置断点。这对于动态加载的程序集或第三方库的调试特别有用。

#### 设置方法

1. 打开断点面板（`Ctrl+Shift+F8`）
2. 点击 "+" 号旁的下拉箭头
3. 选择"函数断点"
4. 输入函数名

#### 支持的语法格式

```
# C# 函数名格式：
MyNamespace.MyClass.MyMethod                    # 完整限定名
MyClass.MyMethod                                # 类名.方法名
MyMethod                                        # 仅方法名（可能有歧义）

# 特殊情况：
Program.Main                                    # 入口方法
Startup.ConfigureServices                       # ASP.NET Core 方法
MyController.GetUser                            # 控制器动作
```

#### 适用场景

```csharp
// 场景1：在不知道具体源码位置的情况下调试
// 函数断点: "Program.Main"
// 可以捕获应用启动时的所有初始化信息

// 场景2：调试中间件管道
// 函数断点: "RequestLoggingMiddleware.InvokeAsync"
// 无需找到中间件文件位置

// 场景3：调试第三方库
// 函数断点: "Newtonsoft.Json.JsonSerializer.Serialize"
// 查看 JSON 序列化过程中的数据
```

### 2.4 数据断点 (Data Breakpoints / Watchpoints)

数据断点在变量的值发生变化时触发，而不是在特定代码行。这在调试意外的状态修改时非常强大。

> **注意**: 数据断点目前主要支持 C/C++ 和部分语言，对 C# 的支持有限制（需要启用特定的调试引擎）。

#### 设置方法

1. 在变量窗口中右键点击要监视的属性
2. 选择"设置数据断点"
3. 或者使用调试控制台命令

#### 使用限制与替代方案

由于 C# 的数据断点支持有限，可以使用以下替代方案：

```csharp
// 替代方案1：使用属性包装器
private int _count;
public int Count
{
    get => _count;
    set
    {
        // 在此设条件断点: value > 100
        System.Diagnostics.Debug.WriteLine(
            $"Count changed from {_count} to {value}");
        _count = value;
    }
}

// 替代方案2：使用 Debugger.Break()
if (someCondition)
{
    System.Diagnostics.Debugger.Break(); // 编程式断点
}

// 替代方案3：使用 Debugger.IsAttached
if (System.Diagnostics.Debugger.IsAttached && someComplexCondition)
{
    // 条件性编程断点
}
```

---

## 三、调试窗口详解

VS Code 的调试界面由多个面板组成，每个面板都有其特定用途。

### 3.1 变量窗口 (Variables Window)

变量窗口显示当前作用域内的所有变量及其值，是最常用的调试面板之一。

#### 局部变量 (Locals) 区域

自动显示当前执行位置的局部变量：

```
┌─ 局部变量 ─────────────────────────────┐
│ 名称          │ 类型           │ 值     │
├───────────────┼────────────────┼────────┤
│ this          │ UserController │ {...}  │
│ id            │ int            │ 42     │
│ user          │ User?          │ null   │
│ cancellationToken│ CancellationToken│ {...} │
│ <返回值>       │ ActionResult   │ (未求值)│
└───────────────┴────────────────┴────────┘
```

#### 监视表达式 (Watch) 区域

可以手动添加想要持续关注的表达式：

```
操作方式：
1. 双击空白区域输入表达式
2. 右键变量 -> "添加到监视"
3. 从编辑器选中表达式 -> 右键 -> "添加到监视"

常用监视表达式：
├── _context.Users.Count()           # 数据库记录数
├── request.Headers["Authorization"] # 请求头
├── HttpContext.User.Identity.Name   # 当前用户
├── Configuration["ConnectionStrings:Default"] # 连接字符串
└── DateTime.Now - requestStartTime  # 请求耗时
```

#### 高级监视技巧

```csharp
// 1. 监视复杂表达式
// 输入: users.Where(u => u.IsActive).Count()

// 2. 调用方法（注意副作用）
// 输入: user.GetDisplayName()

// 3. LINQ 查询预览
// 输入: users.Take(5).Select(u => new { u.Name, u.Email })

// 4. 使用 $ 字符串格式化
// 输入: $"User {user.Name} has {user.Orders.Count} orders"

// 5. 条件表达式
// 输入: user.Age >= 18 ? "成年" : "未成年"
```

#### 调用堆栈 (Call Stack) 区域

显示当前的函数调用链，帮助理解代码执行路径：

```
调用堆栈示例：
▼ [0] UserController.GetUserAsync (UsersController.cs:45)
    - 当前正在执行的代码位置
  [1] MvcCoreMiddleware.Invoke (MvcCoreMiddleware.cs:52)
    - MVC 中间件调用了控制器
  [2] EndpointMiddleware.Invoke (EndpointMiddleware.cs:30)
    - 端点路由匹配后的调用
  [3] RequestDelegatePipeline.MoveNext (...)
    - 管道中的下一个委托
  [4] HostingApplication.ProcessRequestAsync (HostingApplication.cs:35)
    - Kestrel 处理请求入口
```

**实用操作**：

- **跳转到堆栈帧**：双击任意层级可以查看该位置的变量状态
- **切换上下文**：在堆栈帧之间切换可以查看不同作用域的变量值
- **分析调用来源**：理解是谁调用了当前方法

### 3.2 断点窗口 (Breakpoints Window)

集中管理所有断点的面板。

#### 功能概览

```
┌─ 断点 ──────────────────────────────────────────┐
│ ☑ 文件中的断点                                   │
│   ☑ Program.cs:15 (条件: i == 100)               │
│   ☑ UserService.cs:42 (日志点)                   │
│   ☐ Startup.cs:30 (已禁用)                        │
│                                                   │
│ ☑ 函数断点                                       │
│   ☑ MyService.ProcessData                        │
│                                                   │
│ ☑ 数据断点                                       │
│   ☑ _userData.Count                              │
│                                                   │
│ 操作：                                           │
│ [全部启用] [全部禁用] [移除所有]                  │
└───────────────────────────────────────────────────┘
```

#### 断点管理技巧

```
批量操作：
- Ctrl+A 全选断点 → 批量启用/禁用
- 右键断点 → 复制/粘贴/导出
- 使用断点组（通过标签组织）

断点标签（Label）：
- 为相关断点添加标签，便于分组管理
- 例如："API调试"、"数据库"、"认证流程"
- 可以按标签批量启用/禁用

断点导出/导入：
- 用于团队共享调试配置
- 或在不同环境间同步断点设置
```

### 3.3 调试控制台 (Debug Console)

调试控制台提供了 REPL（Read-Eval-Print Loop）环境，可以在调试过程中交互式地执行代码。

#### 基本用法

```
# 直接输入表达式进行求值
> user.Name
"张三"

> users.Count
142

> DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
"2026-04-17 14:30:00"
```

#### 高级用法

```csharp
// 1. 修改变量值（临时修复测试）
> id = 999
999
// 程序会继续使用新值执行

// 2. 调用方法
> user.IsValid()
true

// 3. 执行多行语句
> var temp = users.Where(u => u.IsActive);
> temp.Count()
85

// 4. 导入命名空间（如果尚未导入）
// 注意：某些情况下可能受限

// 5. 测试修复方案
> var fixedEmail = user.Email?.Trim().ToLower();
> fixedEmail
"user@example.com"
```

### 3.4 其他调试面板

#### 加载模块 (Loaded Modules)

显示当前加载的所有程序集（DLL），对于排查依赖问题很有用：

```
# 常见用途：
# 1. 确认某个 NuGet 包是否正确加载
# 2. 检查程序集版本冲突
# 3. 查看动态加载的程序集
```

#### 线程 (Threads)

多线程调试时查看和管理线程：

```
# 多线程调试要点：
# 1. 冻结/解冻线程（暂停某些线程的执行）
# 2. 切换活动线程查看不同线程的状态
# 3. 查看线程名称和 ID
```

---

## 四、launch.json 配置详解

`launch.json` 是 VS Code 调试的核心配置文件，定义了如何启动和调试应用程序。

### 4.1 基本结构

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      // 配置项...
    }
  ],
  "compounds": []  // 复合配置（多目标调试）
}
```

### 4.2 ASP.NET Core 调试配置

#### 标准 Web 应用配置

```json
{
  "name": ".NET Core Launch (web)",
  "type": "coreclr",
  "request": "launch",
  "preLaunchTask": "build",  // 调试前先执行 build 任务
  
  // 程序启动路径
  "program": "${workspaceFolder}/bin/Debug/net8.0/MyApp.dll",
  
  // 工作目录
  "cwd": "${workspaceFolder}",
  
  // 启动浏览器
  "serverReadyAction": {
    "action": "openExternally",
    "pattern": "\\bNow listening on:\\s+(https?://\\S+)"
  },
  
  // 启动参数（等效于命令行参数）
  "args": [],
  
  // 环境变量
  "env": {
    "ASPNETCORE_ENVIRONMENT": "Development",
    "ASPNETCORE_URLS": "https://localhost:5001;http://localhost:5000"
  },
  
  // 控制台输出目标
  "console": "integratedTerminal",
  
  // 停止调试时行为
  "stopAtEntry": false,
  
  // 启动浏览器时的 URL
  "launchBrowser": {
    "enabled": true,
    "args": "${auto-detect-url}",
    "windows": {
      "command": "cmd.exe",
      "/C": "start ${auto-detect-url}"
    },
    "osx": {
      "command": "open"
    },
    "linux": {
      "command": "xdg-open"
    }
  }
}
```

#### 控制台应用程序配置

```json
{
  "name": ".NET Core Launch (console)",
  "type": "coreclr",
  "request": "launch",
  "preLaunchTask": "build",
  "program": "${workspaceFolder}/bin/Debug/net8.0/ConsoleApp.dll",
  "cwd": "${workspaceFolder}",
  "console": "integratedTerminal",
  "stopAtEntry": false
}
```

#### 附加到进程配置

```json
{
  "name": ".NET Core Attach to Process",
  "type": "coreclr",
  "request": "attach",
  "processId": "${command:pickProcess}"
  // processId 也可以指定为具体的 PID
  // "processId": 12345
}
```

### 4.3 多目标调试（复合配置）

复合配置允许同时启动多个调试会话，例如前端 + 后端同时调试。

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": ".NET Core Backend",
      "type": "coreclr",
      "request": "launch",
      "program": "${workspaceFolder}/Api/bin/Debug/net8.0/Api.dll",
      "env": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "cwd": "${workspaceFolder}/Api"
    },
    {
      "name": "Launch Chrome",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:5000",
      "webRoot": "${workspaceFolder}/WebApp/wwwroot",
      "sourceMaps": true
    }
  ],
  "compounds": [
    {
      "name": "Full Stack Debug (API + Frontend)",
      "configurations": [".NET Core Backend", "Launch Chrome"],
      "preLaunchTask": "build-all",
      "stopAll": true  // 停止任一会话时停止所有
    }
  ]
}
```

### 4.4 Docker 容器内调试

远程调试容器中的应用程序：

```json
{
  "name": "Docker .NET Core Attach",
  "type": "coreclr",
  "request": "attach",
  "processId": "${command:pickRemoteProcess}",
  "pipeTransport": {
    "pipeProgram": "docker",
    "pipeArgs": ["exec", "-i", "my-container-name"],
    "debuggerPath": "/vsdbg/vsdbg",
    "pipeCwd": "/app",
    "quoteArgs": false
  },
  "sourceFileMap": {
    "/app": "${workspaceFolder}"
  }
}
```

Dockerfile 中需要安装调试器：

```dockerfile
# 安装 .NET 调试器
RUN curl -sSL https://aka.ms/getvsdbgsh | /bin/sh \
    -s -- --remove-after-install --install-dir /vsdbg \
    -v latest --retry 10
```

### 4.5 高级配置选项

```json
{
  // 其他有用配置项：

  // 源代码映射（用于符号服务器等场景）
  "sourceFileMap": {
    "/src/project": "${workspaceFolder}"
  },

  // 符号路径
  "symbolOptions": {
    "searchPaths": [
      "${workspaceFolder}/symbols",
      "C:\\Symbols"
    ],
    "searchMicrosoftSymbolServer": true,
    "searchNuGetOrgSymbolServer": true
  },

  // 跳过特定代码（如库代码）
  "justMyCode": true,

  // 启用 StepOver 过滤（单步跳过特定方法）
  "stepFiltering": true,

  // 异常设置
  "suppressJITOptimizations": true,  // JIT 优化可能影响调试体验

  // 日志级别
  "logging": {
    "exceptions": true,
    "moduleLoad": true,
    "programOutput": true,
    "threadExit": true,
    "engineLogging": false
  }
}
```

### 4.6 变量替换

VS Code 支持多种内置变量用于动态配置：

| 变量 | 说明 | 示例值 |
|------|------|--------|
| `${workspaceFolder}` | 工作区根目录 | `f:\code\myproject` |
| `${file}` | 当前打开的文件 | `...\Program.cs` |
| `${fileBasename}` | 文件名（含扩展名） | `Program.cs` |
| `${fileBasenameNoExtension}` | 文件名（不含扩展名） | `Program` |
| `${fileDirname}` | 文件所在目录 | `...\src\` |
| `${relativeFile}` | 相对于工作区的路径 | `src\Program.cs` |
| `${pathSeparator}` | 路径分隔符 | `\` 或 `/` |

---

## 五、ASP.NET Core 常见调试场景

### 5.1 API 请求调试

调试 RESTful API 是日常开发中最常见的场景。

#### 完整调试流程

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly ILogger<ProductsController> _logger;

    // ========== 断点设置策略 ==========

    // 断点1：入口处 - 检查请求参数
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts(
        [FromQuery] string? category,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        // F9: 检查 category, page, pageSize 参数值
        _logger.LogInformation("获取产品列表: Category={Category}", category);

        // 断点2：查询构建前 - 检查过滤条件
        IQueryable<Product> query = _context.Products;

        if (!string.IsNullOrEmpty(category))
        {
            query = query.Where(p => p.Category == category);
        }

        // 断点3：分页前 - 检查总数量
        var totalCount = await query.CountAsync();

        // 断点4：结果返回前 - 检查 DTO 映射
        var products = await query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price
            })
            .ToListAsync();

        return Ok(new PagedResult<ProductDto>
        {
            Items = products,
            TotalCount = totalCount,
            Page = page,
            PageSize = pageSize
        });
    }

    // ========== POST 请求调试 ==========

    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct(
        CreateProductRequest request)
    {
        // 日志断点（不暂停）：记录接收到的请求数据
        // 消息: "CreateProduct: Name={request.Name}, Price={request.Price}"

        // 条件断点：只在价格异常时暂停
        // 条件: request.Price <= 0 || request.Price > 100000

        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            Category = request.Category
        };

        _context.Products.Add(product);

        try
        {
            // 断点：保存变更前后
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateException ex)
        {
            // 断点：异常处理分支
            _logger.LogError(ex, "保存产品失败");
            throw;
        }

        // 断点：返回前检查 CreatedAtAction 的路由
        return CreatedAtAction(
            nameof(GetProduct),
            new { id = product.Id },
            ProductDto.FromEntity(product));
    }
}
```

#### 调试技巧

```csharp
// 技巧1：使用 Thunder Client 或 REST Client 扩展发送测试请求
// 这样可以在保持调试会话的同时多次发送请求

// 技巧2：检查 ModelState
// 在 Action 开头设断点，检查 ModelState.Values
// 查看哪些字段验证失败及错误信息

// 技巧3：查看原始请求体
// 在调试控制台中执行：
> Request.Body  // 或通过 HttpContext.Request

// 技巧4：跟踪 Entity Framework 生成的 SQL
// 在 DbContext 中启用敏感数据日志记录：
optionsBuilder.EnableSensitiveDataLogging();
optionsBuilder.EnableDetailedErrors();
// 然后在 Output 窗口筛选 "Microsoft.EntityFrameworkCore" 输出
```

### 5.2 中间件调试

中间件是 ASP.NET Core 请求管道的核心组件，正确调试中间件至关重要。

```csharp
// 自定义中间件调试
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(RequestDelegate next,
        ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        
        // 断点A：请求进入中间件
        var path = context.Request.Path.Value;
        var method = context.Request.Method;
        
        // 日志断点：记录请求开始
        // "→ [{method}] {path}"

        try
        {
            await _next(context);  // F11：可以进入后续中间件
            
            // 断点B：请求完成
            stopwatch.Stop();
            
            // 日志断点：记录响应时间和状态码
            // "← [{method}] {path} - {context.Response.StatusCode} ({stopwatch.ElapsedMilliseconds}ms)"
            
            // 条件断点：慢请求告警
            // 条件: stopwatch.ElapsedMilliseconds > 1000
        }
        catch (Exception ex)
        {
            // 断点C：异常捕获
            stopwatch.Stop();
            _logger.LogError(ex, "请求处理异常: {Path}", path);
            throw; // Shift+F11：跳出后查看谁处理了这个异常
        }
    }
}
```

#### 中间件管道可视化调试

```csharp
// 在 Program.cs 或 Startup.cs 中注册中间件时
var app = builder.Build();

// 使用断点和 F11 单步进入来追踪请求流经每个中间件的顺序
app.UseHttpsRedirection();         // 断点1
app.UseStaticFiles();               // 断点2
app.UseRouting();                   // 断点3
app.UseAuthentication();            // 断点4
app.UseAuthorization();             // 断点5
app.MapControllers();               // 断点6
app.Run();                          // 终端中间件

// 调试顺序建议：
// 1. 在 UseRouting 设断点，确认路由匹配
// 2. 在 UseAuthentication 设断点，检查认证信息
// 3. 在 Controller Action 设断点，检查业务逻辑
```

### 5.3 异步代码调试

异步代码的调试需要特别注意执行上下文和线程切换。

```csharp
public async Task<OrderSummary> GetOrderSummaryAsync(int orderId)
{
    // 断点1：异步方法入口
    _logger.LogInformation("开始获取订单摘要: {OrderId}", orderId);
    
    // 关键：await 前后的代码可能在不同的线程上执行
    var order = await _orderRepository.GetByIdAsync(orderId);
    // F11 进入 await 后，注意调用堆栈的变化
    
    // 断点2：第一个 await 之后
    // 检查 order 对象的状态
    
    // 并行异步操作
    var itemsTask = _orderItemRepository.GetByOrderIdAsync(orderId);
    var customerTask = _customerRepository.GetByIdAsync(order.CustomerId);
    
    // 断点3：并行任务等待前
    // 此时两个任务可能都在执行
    
    await Task.WhenAll(itemsTask, customerTask);
    // 断点4：两个任务都完成后
    
    var items = itemsTask.Result;
    var customer = customerTask.Result;
    
    // 断点5：组装结果
    return new OrderSummary
    {
        Order = order,
        Items = items,
        Customer = customer,
        TotalAmount = items.Sum(i => i.Quantity * i.UnitPrice)
    };
}
```

#### 异步调试注意事项

```
1. 调用堆栈变化
   - await 之后的调用堆栈可能与之前不同
   - 使用 [CallerLineNumber] 等属性帮助追踪

2. 死锁风险
   - 调试时注意 .Result 或 .Wait() 可能导致死锁
   - 优先使用 await 而不是阻塞等待

3. 任务状态检查
   - 在监视窗口查看 Task 的 Status 属性
   - IsCompleted / IsFaulted / IsCanceled

4. 异步上下文
   - 检查 SynchronizationContext.Current
   - ASP.NET Core 默认使用空上下文（ThreadPool）
```

### 5.4 数据库查询调试

Entity Framework Core 的查询调试需要特殊技巧。

```csharp
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;

    public async Task<User?> GetByEmailAsync(string email)
    {
        // 方式1：使用 ToList() 强制执行并查看 SQL
        // 在监视窗口输入：
        // _context.Users.Where(u => u.Email == email).ToList()
        // 然后查看 Output 窗口的 EF 日志
        
        // 方式2：使用 AsQueryable().Expression 查看表达式树
        var query = _context.Users
            .Include(u => u.Profile)
            .Include(u => u.Roles)
            .Where(u => u.Email == email);
        
        // 断点：检查查询表达式
        // 监视: query.Expression.ToString()
        // 可以看到完整的 LINQ 表达式
        
        // 方式3：使用 EF Core 日志
        var result = await query.FirstOrDefaultAsync();
        
        return result;
    }
}
```

#### EF Core 调试配置

```csharp
// Program.cs 中配置 EF Core 详细日志
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(connectionString);
    
    // 开发环境启用详细日志
#if DEBUG
    options.EnableSensitiveDataLogging();
    options.EnableDetailedErrors();
    options.LogTo(
        action => System.Diagnostics.Debug.WriteLine(action),
        LogLevel.Information
    );
#endif
});
```

然后在 VS Code 的 **Output** 面板中选择 **Debug** 或 **.NET Core Log** 来查看 SQL 输出：

```
info: 12/17/2026 10:30:45.123 CoreEventId.ExecutedCommand[20101]
      Executed DbCommand (15ms) [Parameters=[@__email_0='?'], 
      CommandType='Text', CommandTimeout='30']
      SELECT [u].[Id], [u].[Email], [u].[Name], [p].[UserId], 
             [p].[AvatarUrl], [p].[Bio], [r].[UserId], [r].[RoleId]
      FROM [Users] AS [u]
      LEFT JOIN [UserProfiles] AS [p] ON [u].[Id] = [p].[UserId]
      LEFT JOIN [UserRole] AS [r] ON [u].[Id] = [r].[UserId]
      WHERE [u].[Email] = @__email_0
      LIMIT 1
```

---

## 六、调试最佳实践

### 6.1 高效调试工作流

```
高效调试五步法：

1. 复现问题
   - 明确问题的触发条件和预期行为
   - 尽量简化复现步骤

2. 定位可疑代码
   - 通过日志/错误信息缩小范围
   - 使用二分法快速定位（bisect）

3. 设置战略性断点
   - 不要到处设断点
   - 在关键路径上设置条件断点
   - 使用日志断点收集信息而不中断

4. 收集证据
   - 记录变量值、调用堆栈、时间线
   - 使用截图或笔记记录发现

5. 分析与验证
   - 形成假设并通过实验验证
   - 修复后确认问题解决且无副作用
```

### 6.2 性能调试建议

```csharp
// 使用 Stopwatch 进行粗粒度性能测量
public async Task<IActionResult> SlowEndpoint()
{
    var sw = Stopwatch.StartNew();
    
    // 断点1：操作开始
    var data = await _service.ExpensiveOperationAsync();
    
    sw.Stop();
    
    // 条件断点：性能告警
    // 条件: sw.ElapsedMilliseconds > 500
    // 日志消息: "⚠ 慢操作耗时: {sw.ElapsedMilliseconds}ms"
    
    return Ok(data);
}

// 对于更详细的性能分析，考虑使用 dotnet-trace 工具
```

### 6.3 调试安全建议

```
安全注意事项：

1. 敏感数据处理
   - 调试时避免在日志/控制台输出密码、Token 等
   - 生产环境禁用详细错误页面

2. 连接字符串保护
   - 不要硬编码连接字符串
   - 使用 User Secrets 或 Key Vault

3. 调试端口保护
   - 远程调试仅限受信任的网络
   - 使用 VPN 或防火墙规则限制访问

4. 调试完成后清理
   - 移除所有 Debugger.Break() 调用
   - 移除或禁用生产环境的调试配置
   - 清理日志断点
```

### 6.4 常见调试问题排查

| 问题现象 | 可能原因 | 解决方案 |
|----------|----------|----------|
| 断点不被命中 | 代码未重新编译 | 重新构建项目 (`dotnet build`) |
| 断点显示空心圆 | 符号未加载 | 检查构建配置是否为 Debug |
| 变量值无法查看 | 优化代码已被优化 | 设置 `suppressJITOptimizations: true` |
| 无法附加到进程 | 权限不足 | 以管理员身份运行 VS Code |
| 条件断点不起作用 | 表达式语法错误 | 检查条件表达式是否有效 |
| 异步代码断点跳过 | await 后上下文丢失 | 在 await 前后分别设断点 |
| Docker 内无法调试 | vsdbg 未安装 | 更新 Dockerfile 安装调试器 |

---

## 总结

掌握 VS Code 的调试能力将极大提升你的开发效率和问题定位能力。本文涵盖了从基础断点到高级调试技术的完整内容：

- **核心快捷键**：F5/F9/F10/F11/Shift+F11 是调试的基石
- **高级断点**：条件断点、日志断点、函数断点让调试更加精准
- **launch.json**：灵活配置适应各种调试场景
- **专项调试**：API、中间件、异步代码、数据库各有技巧

建议在日常开发中逐步实践这些技巧，形成自己的调试工具箱。
