# Todo应用 - 步骤2：CRUD功能实现

## 一、概述

本步骤将实现Todo应用的核心功能 - **增删改查（CRUD）操作**。这是整个项目最关键的部分，包括：

- ✅ Repository层：数据访问封装
- ✅ Service层：业务逻辑处理
- ✅ Controller层：HTTP请求处理
- ✅ ViewModel层：视图数据传递
- ✅ Razor视图：用户界面
- ✅ 单元测试：代码质量保障

**预计耗时：** 2-3小时

**前置条件：**
- 已完成[步骤1 - 项目初始化](./03-步骤1-项目初始化.md)
- 数据库已创建并包含种子数据
- 项目能够正常编译和运行

---

## 二、架构设计

### 2.1 分层架构数据流

```
浏览器 (Browser)
    ↓ HTTP Request
┌─────────────────┐
│   Controller     │ ← 接收请求、参数验证、调用Service
│  (TodoController) │
└────────┬────────┘
         ↓
┌─────────────────┐
│     Service      │ ← 业务逻辑、事务管理、调用Repository
│   (TodoService)  │
└────────┬────────┘
         ↓
┌─────────────────┐
│   Repository     │ ← EF Core操作数据库、LINQ查询
│ (TodoRepository) │
└────────┬────────┘
         ↓
┌─────────────────┐
│   DbContext      │ ← 数据库连接、变更追踪
│(ApplicationDbContext)
└────────┬────────┘
         ↓
   SQL Server Database
```

### 2.2 职责划分

| 层 | 职责 | 不应该做的事 |
|---|------|------------|
| **Controller** | 接收HTTP请求、参数绑定、返回视图 | 包含复杂业务逻辑 |
| **Service** | 业务规则、数据校验、事务协调 | 直接操作HttpContext |
| **Repository** | 数据库CRUD、查询构造 | 业务规则判断 |
| **ViewModel** | 视图展示所需的数据结构 | 包含行为方法 |

---

## 三、Repository层实现

### 3.1 定义接口

**文件位置：** `src/TodoApp.Core/Interfaces/ITodoRepository.cs`

```csharp
using TodoApp.Core.Entities;

namespace TodoApp.Core.Interfaces
{
    /// <summary>
    /// 待办事项仓储接口
    /// 定义数据访问操作的契约
    /// </summary>
    public interface ITodoRepository
    {
        #region 查询操作

        /// <summary>
        /// 获取所有待办事项（异步）
        /// </summary>
        /// <returns>待办事项列表</returns>
        Task<IEnumerable<TodoItem>> GetAllAsync();

        /// <summary>
        /// 根据ID获取待办事项（异步）
        /// </summary>
        /// <param name="id">待办事项ID</param>
        /// <returns>待办对象，未找到返回null</returns>
        Task<TodoItem?> GetByIdAsync(int id);

        /// <summary>
        /// 根据条件查询待办事项（异步）
        /// </summary>
        /// <param name="predicate">查询条件Lambda表达式</param>
        /// <returns>符合条件的待办事项列表</returns>
        Task<IEnumerable<TodoItem>> FindAsync(Func<TodoItem, bool> predicate);

        #endregion

        #region 写入操作

        /// <summary>
        /// 添加新的待办事项（异步）
        /// </summary>
        /// <param name="todoItem">待添加的实体</param>
        /// <returns>添加后的实体（包含自增ID）</returns>
        Task<TodoItem> AddAsync(TodoItem todoItem);

        /// <summary>
        /// 更新待办事项（异步）
        /// </summary>
        /// <param name="todoItem">要更新的实体</param>
        /// <returns>任务完成状态</returns>
        Task UpdateAsync(TodoItem todoItem);

        /// <summary>
        /// 删除待办事项（异步）
        /// </summary>
        /// <param name="id">要删除的实体ID</param>
        /// <returns>是否删除成功</returns>
        Task<bool> DeleteAsync(int id);

        #endregion

        #region 辅助方法

        /// <summary>
        /// 保存更改到数据库（异步）
        /// </summary>
        /// <returns>受影响的行数</returns>
        Task<int> SaveChangesAsync();

        /// <summary>
        /// 检查指定ID的实体是否存在
        /// </summary>
        /// <param name="id">实体ID</param>
        /// <returns>是否存在</returns>
        Task<bool> ExistsAsync(int id);

        #endregion
    }
}
```

**接口设计原则：**

✅ **面向接口编程**：Controller和Service依赖接口，不依赖具体实现
✅ **单一职责**：每个方法只做一件事
✅ **异步优先**：所有I/O操作都是async/await
✅ **返回值明确**：使用具体的类型而非动态类型

### 3.2 实现Repository类

**文件位置：** `src/TodoApp.Infrastructure/Repositories/TodoRepository.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using TodoApp.Core.Entities;
using TodoApp.Core.Interfaces;
using TodoApp.Infrastructure.Data;

namespace TodoApp.Infrastructure.Repositories
{
    /// <summary>
    /// 待办事项仓储实现
    /// 使用Entity Framework Core进行数据访问
    /// </summary>
    public class TodoRepository : ITodoRepository
    {
        // 注入数据库上下文
        private readonly ApplicationDbContext _context;
        private readonly ILogger<TodoRepository> _logger;

        /// <summary>
        /// 构造函数 - 通过依赖注入接收DbContext
        /// </summary>
        /// <param name="context">数据库上下文</param>
        /// <param name="logger">日志记录器</param>
        public TodoRepository(ApplicationDbContext context, ILogger<TodoRepository> logger)
        {
            _context = context ?? throw new ArgumentNullException(nameof(context));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        #region 查询操作实现

        /// <summary>
        /// 获取所有待办事项
        /// 使用AsNoTracking优化只读查询性能
        /// </summary>
        public async Task<IEnumerable<TodoItem>> GetAllAsync()
        {
            try
            {
                _logger.LogDebug("正在获取所有待办事项");

                // AsNoTracking: 不跟踪实体变化，提升只读查询性能
                var items = await _context.TodoItems
                    .AsNoTracking()  // 性能优化：不进行变更追踪
                    .OrderByDescending(t => t.CreatedAt)  // 按创建时间倒序
                    .ToListAsync();

                _logger.LogDebug("成功获取 {Count} 条待办事项", items.Count);
                return items;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "获取待办事项列表时发生错误");
                throw;  // 重新抛出，由上层处理
            }
        }

        /// <summary>
        /// 根据ID获取单个待办事项
        /// </summary>
        public async Task<TodoItem?> GetByIdAsync(int id)
        {
            try
            {
                _logger.LogDebug("正在获取ID为 {Id} 的待办事项", id);

                var item = await _context.TodoItems
                    .AsNoTracking()
                    .FirstOrDefaultAsync(t => t.Id == id);

                if (item == null)
                {
                    _logger.LogWarning("未找到ID为 {Id} 的待办事项", id);
                }

                return item;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "获取ID为 {Id} 的待办事项时发生错误", id);
                throw;
            }
        }

        /// <summary>
        /// 根据条件查询待办事项
        /// 注意：这里使用客户端评估，对于大数据量应改用表达式树
        /// </summary>
        public async Task<IEnumerable<TodoItem>> FindAsync(Func<TodoItem, bool> predicate)
        {
            try
            {
                _logger.LogDebug("执行条件查询");

                // 在内存中过滤（适用于小数据量）
                var items = await _context.TodoItems
                    .AsNoTracking()
                    .ToListAsync();

                return items.Where(predicate).ToList();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "条件查询时发生错误");
                throw;
            }
        }

        #endregion

        #region 写入操作实现

        /// <summary>
        /// 添加新的待办事项
        /// </summary>
        public async Task<TodoItem> AddAsync(TodoItem todoItem)
        {
            if (todoItem == null)
            {
                throw new ArgumentNullException(nameof(todoItem));
            }

            try
            {
                _logger.LogInformation("正在添加新待办事项: {Title}", todoItem.Title);

                // 设置创建时间
                todoItem.CreatedAt = DateTime.Now;
                todoItem.UpdatedAt = DateTime.Now;

                // 添加到DbContext
                var entityEntry = await _context.TodoItems.AddAsync(todoItem);

                // 立即保存以获取自增ID
                await _context.SaveChangesAsync();

                _logger.LogInformation("成功添加待办事项，ID: {Id}", entityEntry.Entity.Id);
                return entityEntry.Entity;
            }
            catch (DbUpdateException ex)
            {
                _logger.LogError(ex, "添加待办事项时数据库更新失败");
                throw new InvalidOperationException("无法保存待办事项到数据库", ex);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "添加待办事项时发生未知错误");
                throw;
            }
        }

        /// <summary>
        /// 更新待办事项
        /// EF Core会自动检测修改的字段
        /// </summary>
        public async Task UpdateAsync(TodoItem todoItem)
        {
            if (todoItem == null)
            {
                throw new ArgumentNullException(nameof(todoItem));
            }

            try
            {
                _logger.LogInformation("正在更新ID为 {Id} 的待办事项", todoItem.Id);

                // 更新时间戳
                todoItem.UpdatedAt = DateTime.Now;

                // 标记实体为已修改
                _context.TodoItems.Update(todoItem);

                // 保存更改
                await _context.SaveChangesAsync();

                _logger.LogInformation("成功更新待办事项 ID: {Id}", todoItem.Id);
            }
            catch (DbUpdateConcurrencyException ex)
            {
                _logger.LogError(ex, "更新待办事项时发生并发冲突，ID: {Id}", todoItem.Id);
                throw;  // 由Service层处理并发冲突
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "更新待办事项时发生错误，ID: {Id}", todoItem.Id);
                throw;
            }
        }

        /// <summary>
        /// 删除待办事项
        /// 支持物理删除（后续可扩展为软删除）
        /// </summary>
        public async Task<bool> DeleteAsync(int id)
        {
            try
            {
                _logger.LogInformation("正在删除ID为 {Id} 的待办事项", id);

                // 先查找实体
                var todoItem = await _context.TodoItems.FindAsync(id);

                if (todoItem == null)
                {
                    _logger.LogWarning("尝试删除不存在的待办事项，ID: {Id}", id);
                    return false;
                }

                // 从DbContext中移除
                _context.TodoItems.Remove(todoItem);

                // 保存更改
                await _context.SaveChangesAsync();

                _logger.LogInformation("成功删除待办事项 ID: {Id}", id);
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "删除待办事项时发生错误，ID: {Id}", id);
                throw;
            }
        }

        #endregion

        #region 辅助方法实现

        /// <summary>
        /// 保存所有挂起的更改
        /// </summary>
        public async Task<int> SaveChangesAsync()
        {
            return await _context.SaveChangesAsync();
        }

        /// <summary>
        /// 检查实体是否存在
        /// </summary>
        public async Task<bool> ExistsAsync(int id)
        {
            return await _context.TodoItems.AnyAsync(t => t.Id == id);
        }

        #endregion

        #region IDisposable 实现

        /// <summary>
        /// 释放资源
        /// 注意：由于DbContext由DI容器管理生命周期，通常不需要手动释放
        /// 这里仅为演示完整模式
        /// </summary>
        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        protected virtual void Dispose(bool disposing)
        {
            if (disposing)
            {
                // 如果需要手动管理DbContext，在这里释放
                // _context?.Dispose();
            }
        }

        #endregion
    }
}
```

**代码亮点：**

#### 1️⃣ 日志记录
```csharp
_logger.LogInformation("正在添加新待办事项: {Title}", todoItem.Title);
```
- 记录关键操作，便于调试和审计
- 使用结构化日志，支持日志聚合工具

#### 2️⃣ 异常处理策略
```csharp
catch (DbUpdateException ex)
{
    _logger.LogError(ex, "数据库更新失败");
    throw new InvalidOperationException("无法保存数据", ex);
}
```
- 区分不同类型的异常
- 包装异常，隐藏底层技术细节
- 记录完整的错误堆栈

#### 3️⃣ 参数验证
```csharp
if (todoItem == null)
{
    throw new ArgumentNullException(nameof(todoItem));
}
```
- 防御性编程
- 快速失败（Fail Fast）原则

---

## 四、Service层实现

### 4.1 定义服务接口

**文件位置：** `src/TodoApp.Core/Interfaces/ITodoService.cs`

```csharp
using TodoApp.Core.Entities;

namespace TodoApp.Core.Interfaces
{
    /// <summary>
    /// 待办事项服务接口
    /// 定义业务逻辑操作的契约
    /// </summary>
    public interface ITodoService
    {
        #region 查询操作

        /// <summary>
        /// 获取所有待办事项
        /// </summary>
        Task<IEnumerable<TodoItem>> GetAllTodosAsync();

        /// <summary>
        /// 根据ID获取待办事项详情
        /// </summary>
        Task<TodoItem?> GetTodoByIdAsync(int id);

        #endregion

        #region CRUD操作

        /// <summary>
        /// 创建新的待办事项
        /// </summary>
        /// <param name="todoItem">待创建的实体</param>
        /// <returns>创建后的实体</returns>
        Task<TodoItem> CreateTodoAsync(TodoItem todoItem);

        /// <summary>
        /// 更新待办事项
        /// </summary>
        /// <param name="id">待更新的ID</param>
        /// <param name="todoItem">更新数据</param>
        /// <returns>更新后的实体</returns>
        Task<TodoItem> UpdateTodoAsync(int id, TodoItem todoItem);

        /// <summary>
        /// 删除待办事项
        /// </summary>
        /// <param name="id">待删除的ID</param>
        /// <returns>是否删除成功</returns>
        Task<bool> DeleteTodoAsync(int id);

        #endregion

        #region 特殊操作

        /// <summary>
        /// 切换完成状态
        /// </summary>
        /// <param name="id">待办事项ID</param>
        /// <returns>更新后的实体</returns>
        Task<TodoItem> ToggleCompleteAsync(int id);

        #endregion
    }
}
```

### 4.2 实现服务类

**文件位置：** `src/TodoApp.Api/Services/TodoService.cs` （或放在Core项目中）

```csharp
using Microsoft.Extensions.Logging;
using TodoApp.Core.Entities;
using TodoApp.Core.Interfaces;

namespace TodoApp.Api.Services
{
    /// <summary>
    /// 待办事项服务实现
    /// 处理业务逻辑和规则验证
    /// </summary>
    public class TodoService : ITodoService
    {
        private readonly ITodoRepository _repository;
        private readonly ILogger<TodoService> _logger;

        /// <summary>
        /// 构造函数 - 注入依赖
        /// </summary>
        public TodoService(ITodoRepository repository, ILogger<TodoService> logger)
        {
            _repository = repository ?? throw new ArgumentNullException(nameof(repository));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        #region 查询操作

        /// <summary>
        /// 获取所有待办事项
        /// 可在此处添加缓存、权限过滤等逻辑
        /// </summary>
        public async Task<IEnumerable<TodoItem>> GetAllTodosAsync()
        {
            _logger.LogDebug("Service: 获取所有待办事项");

            var todos = await _repository.GetAllAsync();

            // 可以在这里添加业务逻辑，如：
            // - 过滤已软删除的记录
            // - 排序规则
            // - 权限检查

            return todos;
        }

        /// <summary>
        /// 根据ID获取待办事项
        /// </summary>
        public async Task<TodoItem?> GetTodoByIdAsync(int id)
        {
            if (id <= 0)
            {
                throw new ArgumentException("无效的ID", nameof(id));
            }

            _logger.LogDebug("Service: 获取ID为 {Id} 的待办事项", id);

            var todo = await _repository.GetByIdAsync(id);

            if (todo == null)
            {
                _logger.LogWarning("Service: 未找到ID为 {Id} 的待办事项", id);
            }

            return todo;
        }

        #endregion

        #region 创建操作

        /// <summary>
        /// 创建新的待办事项
        /// 包含业务规则验证
        /// </summary>
        public async Task<TodoItem> CreateTodoAsync(TodoItem todoItem)
        {
            // 参数验证
            if (todoItem == null)
            {
                throw new ArgumentNullException(nameof(todoItem));
            }

            // 业务规则1：标题不能为空或纯空格
            if (string.IsNullOrWhiteSpace(todoItem.Title))
            {
                throw new ArgumentException("标题不能为空", nameof(todoItem.Title));
            }

            // 业务规则2：标题长度限制（虽然数据库也有约束，但前端验证更快）
            if (todoItem.Title.Length > 200)
            {
                throw new ArgumentException("标题长度不能超过200个字符", nameof(todoItem.Title));
            }

            // 业务规则3：截止日期如果是过去的时间，给出警告（但不阻止）
            if (todoItem.DueDate.HasValue && todoItem.DueDate.Value.Date < DateTime.Today)
            {
                _logger.LogWarning("用户设置的截止日期 {DueDate} 是过去的时间",
                    todoItem.DueDate.Value.ToString("yyyy-MM-dd"));
                // 可以选择抛出异常或仅记录警告
            }

            // 业务规则4：设置默认值
            if (!todoItem.Priority.HasValue || todoItem.Priority < 0 || todoItem.Priority > 2)
            {
                todoItem.Priority = 0;  // 默认普通优先级
            }

            _logger.LogInformation("Service: 创建新待办事项 '{Title}'", todoItem.Title);

            // 调用Repository保存
            var createdTodo = await _repository.AddAsync(todoItem);

            _logger.LogInformation("Service: 成功创建待办事项，ID: {Id}", createdTodo.Id);

            return createdTodo;
        }

        #endregion

        #region 更新操作

        /// <summary>
        /// 更新待办事项
        /// </summary>
        public async Task<TodoItem> UpdateTodoAsync(int id, TodoItem todoItem)
        {
            // 验证ID一致性
            if (id != todoItem.Id)
            {
                throw new ArgumentException("URL中的ID与实体ID不匹配");
            }

            // 验证实体存在
            var existingTodo = await _repository.GetByIdAsync(id);
            if (existingTodo == null)
            {
                throw new KeyNotFoundException($"未找到ID为 {id} 的待办事项");
            }

            // 业务规则验证（与创建相同）
            if (string.IsNullOrWhiteSpace(todoItem.Title))
            {
                throw new ArgumentException("标题不能为空", nameof(todoItem.Title));
            }

            _logger.LogInformation("Service: 更新待办事项 ID: {Id}, 新标题: '{Title}'",
                id, todoItem.Title);

            // 执行更新
            await _repository.UpdateAsync(todoItem);

            // 返回更新后的数据
            return await _repository.GetByIdAsync(id);
        }

        #endregion

        #region 删除操作

        /// <summary>
        /// 删除待办事项
        /// </summary>
        public async Task<bool> DeleteTodoAsync(int id)
        {
            if (id <= 0)
            {
                throw new ArgumentException("无效的ID", nameof(id));
            }

            // 验证存在性
            var exists = await _repository.ExistsAsync(id);
            if (!exists)
            {
                _logger.LogWarning("Service: 尝试删除不存在的待办事项，ID: {Id}", id);
                return false;
            }

            _logger.LogInformation("Service: 删除待办事项 ID: {Id}", id);

            // 执行删除
            var result = await _repository.DeleteAsync(id);

            return result;
        }

        #endregion

        #region 特殊操作

        /// <summary>
        /// 切换完成状态（快速操作）
        /// 无需进入编辑页面即可切换
        /// </summary>
        public async Task<TodoItem> ToggleCompleteAsync(int id)
        {
            var todo = await _repository.GetByIdAsync(id);
            if (todo == null)
            {
                throw new KeyNotFoundException($"未找到ID为 {id} 的待办事项");
            }

            _logger.LogInformation("Service: 切换待办事项 ID: {Id} 的完成状态，当前: {CurrentStatus}",
                id, todo.IsCompleted);

            // 切换状态
            todo.IsCompleted = !todo.IsCompleted;

            // 保存更改
            await _repository.UpdateAsync(todo);

            _logger.LogInformation("Service: 状态切换完成，新状态: {NewStatus}", todo.IsCompleted);

            return todo;
        }

        #endregion
    }
}
```

**Service层的价值：**

| 关注点 | Repository层 | Service层 |
|--------|-------------|----------|
| **职责** | 数据存取 | 业务规则 |
| **示例** | `SELECT * FROM Todos` | 标题不能为空、日期有效性 |
| **复用性** | 高（被多个Service使用） | 中（特定业务场景） |
| **测试性** | 需要Mock数据库 | 可以独立测试逻辑 |

---

## 五、ViewModel定义

ViewModel是专门为视图设计的模型，与数据库实体分离。

### 5.1 创建ViewModel

**文件位置：** `src/TodoApp.Api/Models/TodoCreateViewModel.cs`

```csharp
using System.ComponentModel.DataAnnotations;

namespace TodoApp.Api.Models
{
    /// <summary>
    /// 创建待办事项的ViewModel
    /// 用于Create页面的表单绑定
    /// </summary>
    public class TodoCreateViewModel
    {
        /// <summary>
        /// 标题（必填）
        /// </summary>
        [Required(ErrorMessage = "标题是必填项")]
        [StringLength(200, MinimumLength = 2, ErrorMessage = "标题长度必须在2-200个字符之间")]
        [Display(Name = "标题")]
        public string Title { get; set; } = string.Empty;

        /// <summary>
        /// 描述（可选）
        /// </summary>
        [StringLength(2000, ErrorMessage = "描述不能超过2000个字符")]
        [Display(Name = "详细描述")]
        public string? Description { get; set; }

        /// <summary>
        /// 优先级
        /// </summary>
        [Display(Name = "优先级")]
        [Range(0, 2, ErrorMessage = "请选择有效的优先级")]
        public int Priority { get; set; } = 0;

        /// <summary>
        /// 截止日期
        /// </summary>
        [Display(Name = "截止日期")]
        public DateTime? DueDate { get; set; }
    }
}
```

**文件位置：** `src/TodoApp.Api/Models/TodoEditViewModel.cs`

```csharp
using System.ComponentModel.DataAnnotations;

namespace TodoApp.Api.Models
{
    /// <summary>
    /// 编辑待办事项的ViewModel
    /// 继承创建ViewModel，增加ID字段
    /// </summary>
    public class TodoEditViewModel
    {
        /// <summary>
        /// 待办事项ID（隐藏字段）
        /// </summary>
        public int Id { get; set; }

        /// <summary>
        /// 标题（必填）
        /// </summary>
        [Required(ErrorMessage = "标题是必填项")]
        [StringLength(200, MinimumLength = 2, ErrorMessage = "标题长度必须在2-200个字符之间")]
        [Display(Name = "标题")]
        public string Title { get; set; } = string.Empty;

        /// <summary>
        /// 描述
        /// </summary>
        [StringLength(2000)]
        [Display(Name = "详细描述")]
        public string? Description { get; set; }

        /// <summary>
        /// 是否完成
        /// </summary>
        [Display(Name = "已完成")]
        public bool IsCompleted { get; set; }

        /// <summary>
        /// 优先级
        /// </summary>
        [Display(Name = "优先级")]
        [Range(0, 2)]
        public int Priority { get; set; } = 0;

        /// <summary>
        /// 截止日期
        /// </summary>
        [Display(Name = "截止日期")]
        public DateTime? DueDate { get; set; }
    }
}
```

**为什么使用ViewModel而不是直接用Entity？**

| 特性 | Entity (TodoItem) | ViewModel |
|------|------------------|-----------|
| **用途** | 数据库映射 | 视图展示 |
| **验证** | 基础约束 | 完整的业务验证 |
| **安全性** | 可能暴露内部字段 | 只暴露需要的字段 |
| **灵活性** | 固定结构 | 可针对不同页面定制 |

---

## 六、Controller实现

### 6.1 创建TodoController

**文件位置：** `src/TodoApp.Api/Controllers/TodoController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using TodoApp.Api.Models;
using TodoApp.Api.Services;
using TodoApp.Core.Entities;
using TodoApp.Core.Interfaces;

namespace TodoApp.Api.Controllers
{
    /// <summary>
    /// 待办事项控制器
    /// 处理所有与待办事项相关的HTTP请求
    /// 路由前缀: /Todo
    /// </summary>
    public class TodoController : Controller
    {
        private readonly ITodoService _todoService;
        private readonly ILogger<TodoController> _logger;

        /// <summary>
        /// 构造函数 - 依赖注入
        /// </summary>
        public TodoController(ITodoService todoService, ILogger<TodoController> logger)
        {
            _todoService = todoService ?? throw new ArgumentNullException(nameof(todoService));
            _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        }

        #region GET: /Todo (列表页)

        /// <summary>
        /// 显示待办事项列表页面
        /// GET: /Todo
        /// </summary>
        [HttpGet]
        public async Task<IActionResult> Index()
        {
            _logger.LogInformation("访问待办事项列表页");

            try
            {
                // 获取所有待办事项
                var todos = await _todoService.GetAllTodosAsync();

                // 返回列表视图
                return View(todos);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "加载待办事项列表时发生错误");
                return View("Error");  // 返回错误页面
            }
        }

        #endregion

        #region GET: /Todo/Details/{id} (详情页)

        /// <summary>
        /// 显示待办事项详情
        /// GET: /Todo/Details/5
        /// </summary>
        [HttpGet]
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return BadRequest();  // 400 Bad Request
            }

            _logger.LogInformation("查看待办事项详情，ID: {Id}", id);

            try
            {
                var todo = await _todoService.GetTodoByIdAsync(id.Value);

                if (todo == null)
                {
                    _logger.LogWarning("未找到ID为 {Id} 的待办事项", id);
                    return NotFound();  // 404 Not Found
                }

                return View(todo);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "加载详情页时发生错误，ID: {Id}", id);
                return View("Error");
            }
        }

        #endregion

        #region GET: /Todo/Create (显示创建表单)

        /// <summary>
        /// 显示创建待办事项的表单页面
        /// GET: /Todo/Create
        /// </summary>
        [HttpGet]
        public IActionResult Create()
        {
            _logger.LogInformation("显示创建待办事项表单");

            // 返回空的ViewModel
            var viewModel = new TodoCreateViewModel();
            return View(viewModel);
        }

        #endregion

        #region POST: /Todo/Create (提交表单)

        /// <summary>
        /// 处理创建待办事项的表单提交
        /// POST: /Todo/Create
        /// </summary>
        [HttpPost]
        [ValidateAntiForgeryToken]  // CSRF防护
        public async Task<IActionResult> Create(TodoCreateViewModel model)
        {
            _logger.LogInformation("提交创建表单，标题: {Title}", model?.Title);

            // 模型验证检查
            if (!ModelState.IsValid)
            {
                _logger.LogWarning("模型验证失败");
                // 验证失败，重新显示表单并显示错误信息
                return View(model);
            }

            try
            {
                // 将ViewModel转换为Entity
                var todoItem = new TodoItem
                {
                    Title = model.Title.Trim(),
                    Description = model.Description?.Trim(),
                    Priority = model.Priority,
                    DueDate = model.DueDate,
                    IsCompleted = false  // 新建的任务默认未完成
                };

                // 调用Service层创建
                var createdTodo = await _todoService.CreateTodoAsync(todoItem);

                // 设置成功消息（TempData在重定向后仍然可用）
                TempData["SuccessMessage"] = $"待办事项 \"{createdTodo.Title}\" 创建成功！";

                _logger.LogInformation("待办事项创建成功，ID: {Id}", createdTodo.Id);

                // 重定向到列表页（PRG模式：Post-Redirect-Get）
                return RedirectToAction(nameof(Index));
            }
            catch (ArgumentException ex)
            {
                // 业务规则验证失败
                _logger.LogWarning(ex, "创建待办事项时业务验证失败");
                ModelState.AddModelError(string.Empty, ex.Message);
                return View(model);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "创建待办事项时发生未知错误");
                ModelState.AddModelError(string.Empty, "创建失败，请稍后重试。");
                return View(model);
            }
        }

        #endregion

        #region GET: /Todo/Edit/{id} (显示编辑表单)

        /// <summary>
        /// 显示编辑待办事项的表单页面
        /// GET: /Todo/Edit/5
        /// </summary>
        [HttpGet]
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null)
            {
                return BadRequest();
            }

            _logger.LogInformation("显示编辑表单，ID: {Id}", id);

            try
            {
                var todo = await _todoService.GetTodoByIdAsync(id.Value);

                if (todo == null)
                {
                    return NotFound();
                }

                // 将Entity转换为EditViewModel
                var viewModel = new TodoEditViewModel
                {
                    Id = todo.Id,
                    Title = todo.Title,
                    Description = todo.Description,
                    IsCompleted = todo.IsCompleted,
                    Priority = todo.Priority,
                    DueDate = todo.DueDate
                };

                return View(viewModel);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "加载编辑页时发生错误，ID: {Id}", id);
                return View("Error");
            }
        }

        #endregion

        #region POST: /Todo/Edit/{id} (提交编辑)

        /// <summary>
        /// 处理编辑待办事项的表单提交
        /// POST: /Todo/Edit/5
        /// </summary>
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, TodoEditViewModel model)
        {
            if (id != model.Id)
            {
                return BadRequest();
            }

            _logger.LogInformation("提交编辑表单，ID: {Id}", id);

            if (!ModelState.IsValid)
            {
                return View(model);
            }

            try
            {
                // 将ViewModel转换为Entity
                var todoItem = new TodoItem
                {
                    Id = model.Id,
                    Title = model.Title.Trim(),
                    Description = model.Description?.Trim(),
                    IsCompleted = model.IsCompleted,
                    Priority = model.Priority,
                    DueDate = model.DueDate
                    // CreatedAt 保持不变，UpdatedAt 由 Service/Repository 自动更新
                };

                // 调用Service层更新
                var updatedTodo = await _todoService.UpdateTodoAsync(id, todoItem);

                TempData["SuccessMessage"] = $"待办事项 \"{updatedTodo.Title}\" 更新成功！";

                _logger.LogInformation("待办事项更新成功，ID: {Id}", id);

                return RedirectToAction(nameof(Index));
            }
            catch (KeyNotFoundException ex)
            {
                _logger.LogWarning(ex, "待办事项不存在，ID: {Id}", id);
                return NotFound();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "更新待办事项时发生错误，ID: {Id}", id);
                ModelState.AddModelError(string.Empty, "更新失败，请稍后重试。");
                return View(model);
            }
        }

        #endregion

        #region GET: /Todo/Delete/{id} (显示删除确认页)

        /// <summary>
        /// 显示删除确认页面
        /// GET: /Todo/Delete/5
        /// </summary>
        [HttpGet]
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null)
            {
                return BadRequest();
            }

            _logger.LogInformation("显示删除确认页，ID: {Id}", id);

            try
            {
                var todo = await _todoService.GetTodoByIdAsync(id.Value);

                if (todo == null)
                {
                    return NotFound();
                }

                return View(todo);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "加载删除确认页时发生错误，ID: {Id}", id);
                return View("Error");
            }
        }

        #endregion

        #region POST: /Todo/Delete/{id} (确认删除)

        /// <summary>
        /// 确认并执行删除操作
        /// POST: /Todo/Delete/5
        /// </summary>
        [HttpPost, ActionName("Delete")]  // ActionName使路由保持一致
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            _logger.LogInformation("确认删除待办事项，ID: {Id}", id);

            try
            {
                var success = await _todoService.DeleteTodoAsync(id);

                if (!success)
                {
                    _logger.LogWarning("删除失败，待办事项可能不存在，ID: {Id}", id);
                    TempData["ErrorMessage"] = "删除失败，该待办事项可能已被删除。";
                }
                else
                {
                    TempData["SuccessMessage"] = "待办事项已成功删除！";
                    _logger.LogInformation("删除成功，ID: {Id}", id);
                }

                return RedirectToAction(nameof(Index));
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "删除待办事项时发生错误，ID: {Id}", id);
                TempData["ErrorMessage"] = "删除失败，请稍后重试。";
                return RedirectToAction(nameof(Index));
            }
        }

        #endregion

        #region POST: /Todo/ToggleComplete/{id} (AJAX切换状态)

        /// <summary>
        /// AJAX方式切换完成状态
        /// POST: /Todo/ToggleComplete/5
        /// </summary>
        [HttpPost]
        public async Task<IActionResult> ToggleComplete(int id)
        {
            try
            {
                var updatedTodo = await _todoService.ToggleCompleteAsync(id);

                // 返回JSON结果供AJAX使用
                return Json(new
                {
                    success = true,
                    isCompleted = updatedTodo.IsCompleted,
                    message = updatedTodo.IsCompleted ? "已标记为完成" : "已标记为未完成"
                });
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "切换状态失败，ID: {Id}", id);
                return Json(new
                {
                    success = false,
                    message = "操作失败，请刷新页面重试"
                });
            }
        }

        #endregion
    }
}
```

**Controller最佳实践：**

✅ **PRG模式**：POST后Redirect到GET，避免重复提交
✅ **ModelState验证**：检查模型绑定结果
✅ **TempData消息**：跨请求传递提示信息
✅ **ValidateAntiForgeryToken**：防止CSRF攻击
✅ **详细日志**：记录关键操作和异常

---

## 七、Razor视图实现

### 7.1 列表页 Index.cshtml

**文件位置：** `src/TodoApp.Api/Views/Todo/Index.cshtml`

```csharp
@model IEnumerable<TodoApp.Core.Entities.TodoItem>

@{
    ViewData["Title"] = "待办事项列表";
}

<!-- 页面标题 -->
<div class="d-flex justify-content-between align-items-center mb-4">
    <h1><i class="bi bi-list-task"></i> 待办事项列表</h1>
    <a asp-action="Create" class="btn btn-primary">
        <i class="bi bi-plus-circle"></i> 新建待办
    </a>
</div>

<!-- 成功/错误消息提示 -->
@if (TempData["SuccessMessage"] != null)
{
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        @TempData["SuccessMessage"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

@if (TempData["ErrorMessage"] != null)
{
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        @TempData["ErrorMessage"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

<!-- 待办事项表格 -->
<div class="card shadow-sm">
    <div class="card-body">
        @if (Model != null && Model.Any())
        {
            <table class="table table-hover table-striped">
                <thead class="table-dark">
                    <tr>
                        <th style="width: 5%">#</th>
                        <th style="width: 25%">标题</th>
                        <th style="width: 15%">状态</th>
                        <th style="width: 10%">优先级</th>
                        <th style="width: 15%">截止日期</th>
                        <th style="width: 15%">创建时间</th>
                        <th style="width: 15%">操作</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach (var item in Model)
                    {
                        <tr class="@(item.IsCompleted ? "table-light text-decoration-line-through" : "")">
                            <td>@item.Id</td>
                            <td>
                                <strong>@item.Title</strong>
                                @if (!string.IsNullOrEmpty(item.Description))
                                {
                                    <br />
                                    <small class="text-muted">
                                        @(item.Description.Length > 50 ?
                                            item.Description.Substring(0, 50) + "..." :
                                            item.Description)
                                    </small>
                                }
                            </td>
                            <td>
                                @if (item.IsCompleted)
                                {
                                    <span class="badge bg-success">已完成</span>
                                }
                                else
                                {
                                    <span class="badge bg-warning text-dark">进行中</span>
                                }
                            </td>
                            <td>
                                @switch (item.Priority)
                                {
                                    case 0:
                                        <span class="badge bg-secondary">普通</span>;
                                        break;
                                    case 1:
                                        <span class="badge bg-info text-dark">重要</span>;
                                        break;
                                    case 2:
                                        <span class="badge bg-danger">紧急</span>;
                                        break;
                                }
                            </td>
                            <td>
                                @if (item.DueDate.HasValue)
                                {
                                    var isOverdue = item.DueDate.Value.Date < DateTime.Today && !item.IsCompleted;
                                    <span class="@(isOverdue ? "text-danger fw-bold" : "")">
                                        @item.DueDate.Value.ToString("yyyy-MM-dd")
                                        @if (isOverdue)
                                        {
                                            <i class="bi bi-exclamation-triangle"></i>
                                        }
                                    </span>
                                }
                                else
                                {
                                    <span class="text-muted">-</span>
                                }
                            </td>
                            <td>
                                <small>@item.CreatedAt.ToString("MM-dd HH:mm")</small>
                            </td>
                            <td>
                                <div class="btn-group btn-group-sm">
                                    <!-- 详情按钮 -->
                                    <a asp-action="Details" asp-route-id="@item.Id"
                                       class="btn btn-info" title="查看详情">
                                        <i class="bi bi-eye"></i>
                                    </a>

                                    <!-- 编辑按钮 -->
                                    <a asp-action="Edit" asp-route-id="@item.Id"
                                       class="btn btn-warning" title="编辑">
                                        <i class="bi bi-pencil"></i>
                                    </a>

                                    <!-- 删除按钮 -->
                                    <a asp-action="Delete" asp-route-id="@item.Id"
                                       class="btn btn-danger" title="删除">
                                        <i class="bi bi-trash"></i>
                                    </a>

                                    <!-- 切换完成状态按钮 (AJAX) -->
                                    <button type="button"
                                            class="btn @(item.IsCompleted ? "btn-outline-success" : "btn-outline-secondary")"
                                            onclick="toggleComplete(@item.Id)"
                                            title="@(item.IsCompleted ? "标记为未完成" : "标记为完成")">
                                        <i class="bi @(item.IsCompleted ? "bi-check-circle-fill" : "bi-circle")"></i>
                                    </button>
                                </div>
                            </td>
                        </tr>
                    }
                </tbody>
            </table>
        }
        else
        {
            <!-- 空状态提示 -->
            <div class="text-center py-5">
                <i class="bi bi-inbox display-1 text-muted"></i>
                <h4 class="mt-3 text-muted">暂无待办事项</h4>
                <p class="text-muted">点击上方"新建待办"按钮添加第一个任务吧！</p>
                <a asp-action="Create" class="btn btn-primary mt-3">
                    <i class="bi bi-plus-lg"></i> 创建第一个待办事项
                </a>
            </div>
        }
    </div>
</div>

@section Scripts {
    <script>
        // AJAX切换完成状态
        function toggleComplete(id) {
            fetch(`/Todo/ToggleComplete/${id}`, {
                method: 'POST',
                headers: {
                    'RequestVerificationToken': document.querySelector('input[name="__RequestVerificationToken"]').value,
                    'Content-Type': 'application/json'
                }
            })
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    // 刷新页面以显示最新状态
                    location.reload();
                } else {
                    alert(data.message || '操作失败');
                }
            })
            .catch(error => {
                console.error('Error:', error);
                alert('网络错误，请重试');
            });
        }
    </script>
}
```

### 7.2 创建页 Create.cshtml

**文件位置：** `src/TodoApp.Api/Views/Todo/Create.cshtml`

```csharp
@model TodoApp.Api.Models.TodoCreateViewModel

@{
    ViewData["Title"] = "新建待办事项";
}

<h1><i class="bi bi-plus-circle-dotted"></i> 新建待办事项</h1>
<hr />

<div class="row">
    <div class="col-md-8">
        <div class="card shadow">
            <div class="card-body">
                <form asp-action="Create">
                    <!-- 防伪令牌 -->
                    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

                    <!-- 标题输入框 -->
                    <div class="mb-3">
                        <label asp-for="Title" class="form-label fw-bold"></label>
                        <input asp-for="Title" class="form-control" placeholder="请输入待办事项标题..." autofocus />
                        <span asp-validation-for="Title" class="text-danger"></span>
                        <small class="form-text text-muted">必填，最多200个字符</small>
                    </div>

                    <!-- 描述文本域 -->
                    <div class="mb-3">
                        <label asp-for="Description" class="form-label fw-bold"></label>
                        <textarea asp-for="Description" class="form-control" rows="4"
                                  placeholder="请输入详细描述（可选）..."></textarea>
                        <span asp-validation-for="Description" class="text-danger"></span>
                    </div>

                    <div class="row">
                        <!-- 优先级下拉框 -->
                        <div class="col-md-6 mb-3">
                            <label asp-for="Priority" class="form-label fw-bold"></label>
                            <select asp-for="Priority" class="form-select"
                                    asp-items="ViewBag.PriorityList">
                                <option value="">-- 请选择 --</option>
                            </select>
                            <span asp-validation-for="Priority" class="text-danger"></span>
                        </div>

                        <!-- 截止日期 -->
                        <div class="col-md-6 mb-3">
                            <label asp-for="DueDate" class="form-label fw-bold"></label>
                            <input asp-for="DueDate" type="date" class="form-control" />
                            <span asp-validation-for="DueDate" class="text-danger"></span>
                        </div>
                    </div>

                    <!-- 提交按钮组 -->
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-save"></i> 保存
                        </button>
                        <a asp-action="Index" class="btn btn-secondary">
                            <i class="bi bi-arrow-left"></i> 返回列表
                        </a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
    }
}
```

### 7.3 编辑页 Edit.cshtml

**文件位置：** `src/TodoApp.Api/Views/Todo/Edit.cshtml`

```csharp
@model TodoApp.Api.Models.TodoEditViewModel

@{
    ViewData["Title"] = "编辑待办事项";
}

<h1><i class="bi bi-pencil-square"></i> 编辑待办事项</h1>
<hr />

<div class="row">
    <div class="col-md-8">
        <div class="card shadow">
            <div class="card-body">
                <form asp-action="Edit">
                    <!-- 隐藏ID字段 -->
                    <input type="hidden" asp-for="Id" />

                    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

                    <!-- 标题 -->
                    <div class="mb-3">
                        <label asp-for="Title" class="form-label fw-bold"></label>
                        <input asp-for="Title" class="form-control" />
                        <span asp-validation-for="Title" class="text-danger"></span>
                    </div>

                    <!-- 描述 -->
                    <div class="mb-3">
                        <label asp-for="Description" class="form-label fw-bold"></label>
                        <textarea asp-for="Description" class="form-control" rows="4"></textarea>
                        <span asp-validation-for="Description" class="text-danger"></span>
                    </div>

                    <!-- 完成状态复选框 -->
                    <div class="mb-3 form-check">
                        <input class="form-check-input" asp-for="IsCompleted" />
                        <label class="form-check-label" asp-for="IsCompleted"></label>
                    </div>

                    <div class="row">
                        <!-- 优先级 -->
                        <div class="col-md-6 mb-3">
                            <label asp-for="Priority" class="form-label fw-bold"></label>
                            <select asp-for="Priority" class="form-select"
                                    asp-items="ViewBag.PriorityList"></select>
                        </div>

                        <!-- 截止日期 -->
                        <div class="col-md-6 mb-3">
                            <label asp-for="DueDate" class="form-label fw-bold"></label>
                            <input asp-for="DueDate" type="date" class="form-control" />
                        </div>
                    </div>

                    <!-- 按钮 -->
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="bi bi-check-lg"></i> 保存更改
                        </button>
                        <a asp-action="Index" class="btn btn-secondary">
                            <i class="bi bi-x-lg"></i> 取消
                        </a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
    }
}
```

### 7.4 删除确认页 Delete.cshtml

**文件位置：** `src/TodoApp.Api/Views/Todo/Delete.cshtml`

```csharp
@model TodoApp.Core.Entities.TodoItem

@{
    ViewData["Title"] = "删除待办事项";
}

<h1 class="text-danger"><i class="bi bi-exclamation-triangle"></i> 删除确认</h1>
<hr />

<h3>您确定要删除这个待办事项吗？</h3>
<div>
    <dl class="row">
        <dt class="col-sm-3">标题：</dt>
        <dd class="col-sm-9">@Model.Title</dd>

        <dt class="col-sm-3">描述：</dt>
        <dd class="col-sm-9">@(Model.Description ?? "无")</dd>

        <dt class="col-sm-3">状态：</dt>
        <dd class="col-sm-9">
            @(Model.IsCompleted ? "已完成" : "未完成")
        </dd>

        <dt class="col-sm-3">优先级：</dt>
        <dd class="col-sm-9">@Model.Priority</dd>

        <dt class="col-sm-3">截止日期：</dt>
        <dd class="col-sm-9">
            @(Model.DueDate?.ToString("yyyy-MM-dd") ?? "未设置")
        </dd>
    </dl>

    <form asp-action="Delete" method="post">
        <input type="hidden" asp-for="Id" />
        <button type="submit" class="btn btn-danger">
            <i class="bi bi-trash"></i> 确认删除
        </button>
        <a asp-action="Index" class="btn btn-secondary">
            <i class="bi bi-arrow-left"></i> 取消
        </a>
    </form>
</div>
```

---

## 八、注册服务到DI容器

### 8.1 修改Program.cs

在 `Program.cs` 中注册服务和仓储：

```csharp
// 在 builder.Services 部分添加：

// --- 注册Repository ---
builder.Services.AddScoped<ITodoRepository, TodoRepository>();

// --- 注册Service ---
builder.Services.AddScoped<ITodoService, TodoService>();
```

**完整的服务注册部分：**

```csharp
// ============================================
// 1️⃣ 配置服务 (Services Configuration)
// ============================================

// MVC服务
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<AutoValidateAntiforgeryTokenAttribute>();
});

// 开发环境运行时编译
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddRazorRuntimeCompilation();
}

// 数据库服务
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection")
    ?? throw new InvalidOperationException("Connection string 'DefaultConnection' not found.");

builder.Services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(connectionString, sqlOptions =>
    {
        sqlOptions.EnableRetryOnFailure(
            maxRetryCount: 3,
            maxRetryDelay: TimeSpan.FromSeconds(30),
            errorNumbersToAdd: null);

        if (builder.Environment.IsDevelopment())
        {
            options.LogTo(Console.WriteLine, LogLevel.Information);
            options.EnableSensitiveDataLogging();
        }
    });
});

// ⭐ 新增：注册Repository和Service
builder.Services.AddScoped<ITodoRepository, TodoRepository>();
builder.Services.AddScoped<ITodoService, TodoService>();
```

**生命周期说明：**

| 生命周期 | 适用场景 | 本项目使用 |
|---------|---------|----------|
| **Transient** | 无状态、轻量级 | - |
| **Scoped** | 每个请求一个实例 | ✅ Repository, Service |
| **Singleton** | 全局唯一实例 | DbContext（通过AddDbContext自动配置） |

---

## 九、单元测试示例

### 9.1 安装测试包

```bash
cd tests/TodoApp.Tests

# Moq框架（模拟依赖）
dotnet add package Moq --version 4.20.*

# EF Core InMemory（内存数据库用于测试）
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 8.0.*
```

### 9.2 编写Service层测试

**文件位置：** `tests/TodoApp.Tests/Services/TodoServiceTests.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using Moq;
using TodoApp.Core.Entities;
using TodoApp.Core.Enums;
using TodoApp.Core.Interfaces;
using TodoApp.Infrastructure.Data;
using Xunit;

namespace TodoApp.Tests.Services
{
    /// <summary>
    /// TodoService 单元测试
    /// 测试核心业务逻辑的正确性
    /// </summary>
    public class TodoServiceTests
    {
        private readonly Mock<ITodoRepository> _mockRepository;
        private readonly TodoService _todoService;

        public TodoServiceTests()
        {
            // 创建Mock对象
            _mockRepository = new Mock<ITodoRepository>();

            // 创建Service实例，注入Mock的Repository
            _todoService = new TodoService(_mockRepository.Object, Mock.Of<ILogger<TodoService>>());
        }

        #region 创建待办事项测试

        [Fact]
        public async Task CreateTodo_WithValidData_ShouldReturnCreatedTodo()
        {
            // Arrange（准备）
            var newTodo = new TodoItem
            {
                Title = "测试任务",
                Description = "这是一个测试",
                Priority = (int)PriorityLevel.Normal
            };

            var expectedTodo = new TodoItem
            {
                Id = 1,
                Title = "测试任务",
                Description = "这是一个测试",
                Priority = 0
            };

            _mockRepository.Setup(r => r.AddAsync(It.IsAny<TodoItem>()))
                          .ReturnsAsync(expectedTodo);

            // Act（执行）
            var result = await _todoService.CreateTodoAsync(newTodo);

            // Assert（断言）
            Assert.NotNull(result);
            Assert.Equal(1, result.Id);
            Assert.Equal("测试任务", result.Title);
            Assert.Equal(0, result.Priority);

            // 验证Repository的AddAsync方法被调用了一次
            _mockRepository.Verify(r => r.AddAsync(It.IsAny<TodoItem>()), Times.Once);
        }

        [Fact]
        public async Task CreateTodo_WithEmptyTitle_ShouldThrowArgumentException()
        {
            // Arrange
            var invalidTodo = new TodoItem { Title = "" };

            // Act & Assert
            var exception = await Assert.ThrowsAsync<ArgumentException>(
                () => _todoService.CreateTodoAsync(invalidTodo));

            Assert.Contains("标题", exception.Message);
        }

        [Fact]
        public async Task CreateTodo_WithTitleTooLong_ShouldThrowArgumentException()
        {
            // Arrange
            var longTitle = new string('A', 201);  // 超过200字符限制
            var invalidTodo = new TodoItem { Title = longTitle };

            // Act & Assert
            await Assert.ThrowsAsync<ArgumentException>(
                () => _todoService.CreateTodoAsync(invalidTodo));
        }

        [Fact]
        public async Task CreateTodo_WithNull_ShouldThrowArgumentNullException()
        {
            // Act & Assert
            await Assert.ThrowsAsync<ArgumentNullException>(
                () => _todoService.CreateTodoAsync(null!));
        }

        #endregion

        #region 获取待办事项测试

        [Fact]
        public async Task GetTodoById_WithValidId_ShouldReturnTodo()
        {
            // Arrange
            var expectedTodo = new TodoItem
            {
                Id = 1,
                Title = "现有任务",
                IsCompleted = false
            };

            _mockRepository.Setup(r => r.GetByIdAsync(1))
                          .ReturnsAsync(expectedTodo);

            // Act
            var result = await _todoService.GetTodoByIdAsync(1);

            // Assert
            Assert.NotNull(result);
            Assert.Equal(1, result.Id);
            Assert.Equal("现有任务", result.Title);
        }

        [Fact]
        public async Task GetTodoById_WithInvalidId_ShouldReturnNull()
        {
            // Arrange
            _mockRepository.Setup(r => r.GetByIdAsync(999))
                          .ReturnsAsync((TodoItem?)null);

            // Act
            var result = await _todoService.GetTodoByIdAsync(999);

            // Assert
            Assert.Null(result);
        }

        [Fact]
        public async Task GetTodoById_WithZeroOrNegativeId_ShouldThrowException()
        {
            // Act & Assert
            await Assert.ThrowsAsync<ArgumentException>(
                () => _todoService.GetTodoByIdAsync(0));

            await Assert.ThrowsAsync<ArgumentException>(
                () => _todoService.GetTodoByIdAsync(-1));
        }

        #endregion

        #region 切换完成状态测试

        [Fact]
        public async Task ToggleComplete_FromIncompleteToComplete_ShouldWork()
        {
            // Arrange
            var existingTodo = new TodoItem
            {
                Id = 1,
                Title = "未完成任务",
                IsCompleted = false
            };

            _mockRepository.Setup(r => r.GetByIdAsync(1))
                          .ReturnsAsync(existingTodo);

            _mockRepository.Setup(r => r.UpdateAsync(It.IsAny<TodoItem>()))
                          .Callback<TodoItem>(t => existingTodo.IsCompleted = t.IsCompleted)
                          .Returns(Task.CompletedTask);

            _mockRepository.Setup(r => r.GetByIdAsync(1))
                          .ReturnsAsync(existingTodo);

            // Act
            var result = await _todoService.ToggleCompleteAsync(1);

            // Assert
            Assert.True(result.IsCompleted);
        }

        #endregion
    }
}
```

### 9.3 运行测试

```bash
# 进入测试项目目录
cd tests/TodoApp.Tests

# 运行所有测试
dotnet test

# 运行指定测试类
dotnet test --filter "FullyQualifiedName~TodoServiceTests"

# 输出详细的测试结果
dotnet test --verbosity normal
```

**预期输出：**
```
测试运行成功。
总测试数: 8
     通过: 8
 总时间: 0.1234 秒
```

---

## 十、运行和验证

### 10.1 启动应用

```bash
cd src/TodoApp.Api
dotnet run
```

打开浏览器访问：http://localhost:5000/Todo

### 10.2 功能验证清单

#### 验证1：列表页
- [ ] 页面显示12条种子数据
- [ ] 表格正确显示所有列
- [ ] 不同状态的视觉区分明显
- [ ] 操作按钮都可见且样式正确

> [!tip] 📸 预期效果 - Todo 列表页
> 页面顶部显示 **"我的待办事项"** 标题，下方为表格展示所有 Todo 项（标题、状态、创建时间、操作按钮），每行有 **编辑** 和 **删除** 按钮

#### 验证2：创建功能
- [ ] 点击"新建待办"跳转到创建页
- [ ] 表单显示所有必需字段
- [ ] 填写有效数据并提交后跳转回列表
- [ ] 列表中出现新建的记录
- [ ] 显示成功提示消息

**截图占位符：**
![创建页截图](images/todo-create.png)

#### 验证3：编辑功能
- [ ] 点击编辑按钮跳转到编辑页
- [ ] 表单预填充现有数据
- [ ] 修改后保存成功
- [ ] 列表中数据已更新

> [!tip] 📸 预期效果 - 编辑页
> 点击编辑按钮后跳转到编辑页面，URL 变更为 `/Todo/Edit/xxx`，表单中 **预填充当前数据**（标题、描述等字段已有值）

#### 验证4：删除功能
- [ ] 点击删除按钮显示确认页
- [ ] 确认删除后记录从列表消失
- [ ] 显示删除成功提示

> [!tip] 📸 预期效果 - 删除确认
> 点击删除按钮弹出 **确认对话框**（"确定要删除此条记录吗？"），确认后列表中该记录消失，顶部显示 **绿色成功提示** "删除成功"

#### 验证5：状态切换
- [ ] 点击完成状态按钮
- [ ] 状态立即改变（无需刷新整个页面）
- [ ] 已完成任务有明显的视觉标识

---

## 十一、常见问题及解决方案

### 问题1：找不到视图

**错误：**
```
InvalidOperationException: The view 'Index' was not found.
```

**解决方案：**
1. 确认视图文件位于正确的路径：`Views/Todo/Index.cshtml`
2. 确认Controller名称与文件夹匹配：`TodoController` → `Views/Todo/`
3. 检查 `_ViewImports.cshtml` 是否存在

---

### 问题2：模型绑定失败

**现象：** 表单提交后数据为null或默认值

**解决方案：**
1. 检查表单字段的 `name` 属性是否与ViewModel属性名匹配
2. 确保使用了 `[FromBody]` 或 `[FromForm]` 特性（MVC通常不需要）
3. 检查ViewModel是否有无参构造函数

---

### 问题3：CSRF验证失败

**错误：**
```
AntiforgeryToken validation failed.
```

**解决方案：**
1. 确认表单中包含了防伪令牌：`@Html.AntiForgeryToken()` 或 `<form>` 标签自动生成
2. AJAX请求需要在Header中包含令牌
3. 检查 `@Html.AntiForgeryToken()` 是否在每个表单中

---

### 问题4：数据库连接字符串错误

**错误：**
```
SqlException: A network-related error occurred...
```

**解决方案：**
1. 检查 `appsettings.json` 中的连接字符串
2. 确认LocalDB实例名正确：`(localdb)\mssqllocaldb`
3. 尝试使用SSMS手动连接验证

---

## 十二、验证点清单

### 12.1 代码完整性

- [ ] ITodoRepository 接口已定义（8个方法）
- [ ] TodoRepository 类已完整实现
- [ ] ITodoService 接口已定义（6个方法）
- [ ] TodoService 类已完整实现
- [ ] TodoCreateViewModel 和 TodoEditViewModel 已创建
- [ ] TodoController 已实现所有Action（Index, Details, Create, Edit, Delete, ToggleComplete）
- [ ] 所有Razor视图已创建（Index, Create, Edit, Delete, Details）

### 12.2 DI配置

- [ ] Program.cs中已注册 `ITodoRepository → TodoRepository`
- [ ] Program.cs中已注册 `ITodoService → TodoService`
- [ ] 生命周期设置为 Scoped

### 12.3 功能验证

- [ ] 列表页能正常显示数据
- [ ] 创建功能完整（GET + POST）
- [ ] 编辑功能完整（GET + POST）
- [ ] 删除功能完整（确认 + 执行）
- [ ] 状态切换功能正常（AJAX）
- [ ] 表单验证生效（必填、长度等）

### 12.4 测试验证

- [ ] 单元测试项目能编译
- [ ] 至少编写了5个以上的测试用例
- [ ] 所有测试都能通过
- [ ] 测试覆盖主要业务场景

---

## 十三、总结

本步骤完成了Todo应用的**核心CRUD功能**：

### 已完成的工作

✅ **Repository层** - 数据访问封装
- 定义了清晰的接口契约
- 实现了完整的CRUD操作
- 添加了日志和异常处理

✅ **Service层** - 业务逻辑处理
- 实现了输入验证和业务规则
- 协调Repository操作
- 提供了清晰的API

✅ **Controller层** - HTTP请求处理
- 实现了RESTful风格的Action
- PRG模式避免重复提交
- TempData消息反馈机制

✅ **ViewModel层** - 视图数据传输
- 与Entity解耦
- 完整的数据注解验证
- 针对不同场景定制

✅ **Razor视图** - 用户界面
- Bootstrap 5美化
- 响应式布局
- 友好的交互体验

✅ **单元测试** - 质量保障
- xUnit + Moq框架
- 覆盖核心业务逻辑
- 保证代码重构安全

### 关键学习点

1. **分层架构的价值** - 职责清晰、易于测试和维护
2. **依赖注入的使用** - 松耦合、便于替换实现
3. **异步编程模式** - async/await提升性能
4. **ViewModel的作用** - 安全性和灵活性
5. **日志的重要性** - 调试和监控必备

### 下一步

接下来将进入**步骤3：表单验证**，深入完善数据完整性保障机制。

---

**文档版本：** v1.0
**创建日期：** 2026-04-16
**最后更新：** 2026-04-16
**作者：** ASP.NET Web Dev Knowledge Base
**状态：** ✅ 完成，可以进入步骤3
