# 基于角色的授权 (RBAC)

> **学习时间**: 约 50 分钟 | **难度**: ⭐⭐⭐ | **前置知识**: 认证授权概念、Identity Framework 基础

---

## 📌 本节目标

深入理解 RBAC（基于角色的访问控制）模型，掌握 ASP.NET Core 中三种角色授权方式的使用场景和实现方法，能够构建完整的 RBAC 权限管理系统。

---

## 一、RBAC 概念详解

### 1.1 核心思想

**RBAC = Role-Based Access Control（基于角色的访问控制）**

核心思想非常简单：**不直接给用户分配权限，而是把权限打包成"角色"，再把角色分配给用户。**

```
┌─────────────────────────────────────────────────────────────┐
│                    RBAC 权限模型                              │
│                                                             │
│   用户(User)          角色(Role)           权限(Permission)    │
│                                                             │
│   ┌──────┐           ┌──────┐            ┌─────────────┐    │
│   │ 张三  │──────────►│ 管理员 │────────────│ 创建/删除/修改 │    │
│   └──────┘           │      │            │ 查看所有数据   │    │
│                      └──────┘            │ 用户管理      │    │
│   ┌──────┐           ┌──────┐            │ 系统设置      │    │
│   │ 李四  │──────────►│ 编辑者 │────────────│ 修改/查看     │    │
│   └──────┘           │      │            │ 发布内容      │    │
│                      └──────┘            └─────────────┘    │
│   ┌──────┐           ┌──────┐                                │
│   │ 王五  │──────────►│ 访客  │────────────│ 仅查看         │    │
│   └──────┘           └──────┘                                │
│                                                             │
│   ✅ 优势：管理方便，只需调整用户的角色即可改变其权限             │
│   ✅ 优势：符合现实世界的组织架构                               │
│   ❌ 局限：粒度较粗，复杂规则难以用角色表达                     │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 生活类比

想象一家公司：

```
公司组织架构（RBAC 的现实映射）:

CEO (超级管理员)
├── 部门经理 (管理员)
│   ├── 高级工程师 (编辑者)
│   │   └── 初级工程师 (普通用户)
│   └── 实习生 (访客)

每个职位（角色）对应不同的权限：
- CEO: 所有权限
- 经理: 部门内所有权限 + 人事审批
- 高级工程师: 代码提交、代码审查
- 初级工程师: 代码提交
- 实习生: 只读访问

新员工入职 → 分配职位（角色）→ 自动获得该职位的所有权限
员工升职 → 更换职位（角色）→ 权限自动变更
```

### 1.3 RBAC 的四种基本模型

```
RBAC0 (基础模型):
用户 ←→ 角色 ←→ 权限

RBAC1 (层级模型):        RBAC2 (约束模型):
在 RBAC0 基础上增加:       在 RBAC0 基础上增加:
├── 角色继承              ├── 互斥角色（不能同时拥有）
│   管理员 > 编辑者 > 用户  ├── 基数限制（最多 N 个管理员）
│   （高级角色继承低级权限）  ├── 先决条件（先有 A 才能有 B）
└── ...                   └── ...

RBAC3 (统一模型) = RBAC1 + RBAC2
```

---

## 二、ASP.NET Core 中的三种角色授权方式

### 2.1 方式一：声明式授权（Declarative）

使用 `[Authorize]` 特性直接声明所需角色，最简单直观。

```csharp
using Microsoft.AspNetCore.Authorization;

public class AdminController : Controller
{
    // 只有 Admin 角色可以访问
    [Authorize(Roles = "Admin")]
    public IActionResult Dashboard()
    {
        return View();
    }

    // Admin 或 Editor 都可以访问
    [Authorize(Roles = "Admin,Editor")]
    public IActionResult ContentManagement()
    {
        return View();
    }

    // 多个角色必须同时满足（需要自定义 Policy 实现）
    // [Authorize(Roles = "Admin,Editor")]  // 这是 OR 关系
}
```

**适用场景**：固定的、简单的角色检查。

**优点**：代码简洁，一目了然。
**缺点**：硬编码角色名，修改需要重新编译；无法表达复杂的 AND/OR 逻辑。

### 2.2 方式二：策略式授权（Policy-based）

将角色检查封装为可复用的策略，更加灵活。

```csharp
// Program.cs - 注册基于角色的策略
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireAdmin", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("RequireEditorOrAbove", policy =>
        policy.RequireRole("Admin", "Editor"));  // OR 关系

    // 更复杂的组合
    options.AddPolicy("RequireManager", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("Department", "技术部"));  // AND 关系
});

// 使用方式
[Authorize(Policy = "RequireAdmin")]
public IActionResult OnlyAdmin() { /* ... */ }

[Authorize(Policy = "RequireEditorOrAbove")]
public IActionResult EditorAndAbove() { /* ... */ }
```

**适用场景**：需要在多处复用相同的角色组合逻辑。

**优点**：集中管理，修改策略定义即可全局生效；支持更复杂的组合条件。
**缺点**：需要在 Program.cs 中预先注册所有策略。

### 2.3 方式三：命令式/程序化授权（Imperative）

在代码中动态调用 `IAuthorizationService` 进行授权判断。

```csharp
public class ArticleService : IArticleService
{
    private readonly IAuthorizationService _authorizationService;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public ArticleService(
        IAuthorizationService authorizationService,
        IHttpContextAccessor httpContextAccessor)
    {
        _authorizationService = authorizationService;
        _httpContextAccessor = httpContextAccessor;
    }

    public async Task<List<Article>> GetFilteredArticlesAsync()
    {
        var user = _httpContextAccessor.HttpContext!.User;

        // 动态判断当前用户是否有管理员权限
        var adminResult = await _authorizationService.AuthorizeAsync(
            user, null, "RequireAdmin");

        if (adminResult.Succeeded)
        {
            // 管理员可以看到所有文章
            return await _articleRepository.GetAllAsync();
        }
        else
        {
            // 普通用户只能看到已发布的文章
            return await _articleRepository.GetPublishedAsync();
        }
    }

    /// <summary>
    /// 在业务逻辑中根据角色返回不同数据
    /// </summary>
    public async Task<ArticleDto> GetArticleWithPermissionInfo(int articleId)
    {
        var article = await _articleRepository.GetByIdAsync(articleId);
        if (article == null) return null!;

        var user = _httpContextAccessor.HttpContext!.User;

        // 检查是否可以编辑
        var canEdit = (await _authorizationService.AuthorizeAsync(
            user, article, "CanEditArticle")).Succeeded;

        // 检查是否可以删除
        var canDelete = (await _authorizationService.AuthorizeAsync(
            user, article, "CanDeleteArticle")).Succeeded;

        return new ArticleDto
        {
            Id = article.Id,
            Title = article.Title,
            Content = canEdit ? article.Content : article.Content[..200] + "...",
            CanEdit = canEdit,
            CanDelete = canDelete
        };
    }
}
```

**适用场景**：
- 需要在 Service 层或 Repository 层进行权限过滤
- 根据权限动态决定返回的数据内容
- 复杂的业务逻辑中嵌入授权判断

**优点**：最大灵活性，可以在任何地方调用。
**缺点**：代码量较多，分散在各处不易统一管理。

### 2.4 三种方式对比总结

| 维度 | 声明式 `[Authorize(Roles)]` | 策略式 `[Authorize(Policy)]` | 命令式 `IAuthorizationService` |
|------|---------------------------|------------------------------|-------------------------------|
| **位置** | Controller/Action 上 | Controller/Action 上 | 任意代码中 |
| **灵活性** | 低 | 中 | 高 |
| **复用性** | 低 | 高 | 中 |
| **维护性** | 差（分散） | 好（集中） | 中 |
| **运行时决策** | 不支持 | 不支持 | 支持 |
| **推荐场景** | 简单固定角色 | 复杂角色组合 | 业务逻辑中的动态判断 |

---

## 三、角色管理的 CRUD 操作

### 3.1 RoleService 完整实现

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using MyApp.Data;
using MyApp.Models;

public interface IRoleService
{
    Task<List<RoleDto>> GetAllRolesAsync();
    Task<RoleDetailDto?> GetRoleByIdAsync(string roleId);
    Task<IdentityResult> CreateRoleAsync(CreateRoleRequest request);
    Task<IdentityResult> UpdateRoleAsync(string roleId, UpdateRoleRequest request);
    Task<IdentityResult> DeleteRoleAsync(string roleId);
    Task<PagedResult<UserSummaryDto>> GetRoleMembersAsync(
        string roleId, int page, int pageSize);
    Task<IdentityResult> AddUserToRoleAsync(string userId, string roleId);
    Task<IdentityResult> RemoveUserFromRoleAsync(string userId, string roleId);
    Task<bool> IsUserInRoleAsync(string userId, string roleName);
}

public class RoleService : IRoleService
{
    private readonly RoleManager<ApplicationRole> _roleManager;
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly ApplicationDbContext _dbContext;
    private readonly ILogger<RoleService> _logger;

    public RoleService(
        RoleManager<ApplicationRole> roleManager,
        UserManager<ApplicationUser> userManager,
        ApplicationDbContext dbContext,
        ILogger<RoleService> logger)
    {
        _roleManager = roleManager;
        _userManager = userManager;
        _dbContext = dbContext;
        _logger = logger;
    }

    /// <summary>
    /// 获取所有角色列表
    /// </summary>
    public async Task<List<RoleDto>> GetAllRolesAsync()
    {
        return await _roleManager.Roles
            .OrderBy(r => r.SortOrder)
            .ThenBy(r => r.Name)
            .Select(r => new RoleDto
            {
                Id = r.Id,
                Name = r.Name!,
                Description = r.Description,
                IsSystemRole = r.IsSystemRole,
                MemberCount = 0  // 后续填充
            })
            .ToListAsync();
    }

    /// <summary>
    /// 获取角色详情（含成员数量）
    /// </summary>
    public async Task<RoleDetailDto?> GetRoleByIdAsync(string roleId)
    {
        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null) return null;

        var members = await _userManager.GetUsersInRoleAsync(role.Name!);
        var claims = await _roleManager.GetClaimsAsync(role);

        return new RoleDetailDto
        {
            Id = role.Id,
            Name = role.Name!,
            Description = role.Description,
            IsSystemRole = role.IsSystemRole,
            SortOrder = role.SortOrder,
            CreatedAt = role.CreatedAt,
            MemberCount = members.Count,
            Members = members.Take(10).Select(u => new UserSummaryDto
            {
                Id = u.Id,
                UserName = u.UserName!,
                Email = u.Email!,
                Nickname = u.Nickname,
                AvatarUrl = u.AvatarUrl
            }).ToList(),
            Claims = claims.Select(c => new ClaimItemDto
            {
                Type = c.Type,
                Value = c.Value
            }).ToList()
        };
    }

    /// <summary>
    /// 创建新角色
    /// </summary>
    public async Task<IdentityResult> CreateRoleAsync(CreateRoleRequest request)
    {
        // 检查是否重名
        if (await _roleManager.RoleExistsAsync(request.Name))
        {
            return IdentityResult.Failed(new IdentityError
            {
                Code = "DuplicateName",
                Description = $"角色名称 '{request.Name}' 已存在"
            });
        }

        var maxSortOrder = await _roleManager.Roles
            .MaxAsync(r => (int?)r.SortOrder) ?? 0;

        var role = new ApplicationRole
        {
            Name = request.Name.Trim(),
            NormalizedName = request.Name.Trim().ToUpperInvariant(),
            Description = request.Description?.Trim(),
            IsSystemRole = false,
            SortOrder = maxSortOrder + 1,
            CreatedAt = DateTime.UtcNow
        };

        var result = await _roleManager.CreateAsync(role);

        if (result.Succeeded)
        {
            _logger.LogInformation("创建角色成功: {RoleName} (ID: {RoleId})",
                role.Name, role.Id);
        }

        return result;
    }

    /// <summary>
    /// 更新角色信息
    /// </summary>
    public async Task<IdentityResult> UpdateRoleAsync(string roleId, UpdateRoleRequest request)
    {
        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null)
            return IdentityResult.Failed(new IdentityError { Description = "角色不存在" });

        // 如果修改了名称，检查新名称是否冲突
        if (!string.Equals(role.Name, request.Name, StringComparison.OrdinalIgnoreCase))
        {
            if (await _roleManager.RoleExistsAsync(request.Name))
            {
                return IdentityResult.Failed(new IdentityError
                {
                    Code = "DuplicateName",
                    Description = $"角色名称 '{request.Name}' 已存在"
                });
            }

            role.Name = request.Name.Trim();
            role.NormalizedName = request.Name.Trim().ToUpperInvariant();
        }

        role.Description = request.Description?.Trim();
        role.SortOrder = request.SortOrder;

        var result = await _roleManager.UpdateAsync(role);

        if (result.Succeeded)
        {
            _logger.LogInformation("更新角色: {RoleId}, 新名称: {NewName}",
                roleId, role.Name);
        }

        return result;
    }

    /// <summary>
    /// 删除角色
    /// </summary>
    public async Task<IdentityResult> DeleteRoleAsync(string roleId)
    {
        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null)
            return IdentityResult.Failed(new IdentityError { Description = "角色不存在" });

        // 保护系统内置角色
        if (role.IsSystemRole)
        {
            return IdentityResult.Failed(new IdentityError
            {
                Code = "CannotDeleteSystemRole",
                Description = "不能删除系统内置角色"
            });
        }

        // 检查是否有用户在使用此角色
        var usersInRole = await _userManager.GetUsersInRoleAsync(role.Name!);
        if (usersInRole.Any())
        {
            return IdentityResult.Failed(new IdentityError
            {
                Code = "RoleHasMembers",
                Description = $"无法删除角色 '{role.Name}'，仍有 {usersInRole.Count} 个用户在使用"
            });
        }

        var result = await _roleManager.DeleteAsync(role);

        if (result.Succeeded)
        {
            _logger.LogInformation("删除角色: {RoleName}", role.Name);
        }

        return result;
    }

    /// <summary>
    /// 获取角色成员（分页）
    /// </summary>
    public async Task<PagedResult<UserSummaryDto>> GetRoleMembersAsync(
        string roleId, int page, int pageSize)
    {
        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null)
            return new PagedResult<UserSummaryDto>(new(), 0, page, pageSize);

        var allUsers = await _userManager.GetUsersInRoleAsync(role.Name!);
        var totalCount = allUsers.Count;

        var users = allUsers
            .OrderByDescending(u => u.CreatedAt)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(u => new UserSummaryDto
            {
                Id = u.Id,
                UserName = u.UserName!,
                Email = u.Email!,
                Nickname = u.Nickname,
                AvatarUrl = u.AvatarUrl
            })
            .ToList();

        return new PagedResult<UserSummaryDto>(users, totalCount, page, pageSize);
    }

    /// <summary>
    /// 将用户添加到角色
    /// </summary>
    public async Task<IdentityResult> AddUserToRoleAsync(string userId, string roleId)
    {
        var user = await _userManager.FindByIdAsync(userId);
        if (user == null)
            return IdentityResult.Failed(new IdentityError { Description = "用户不存在" });

        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null)
            return IdentityResult.Failed(new IdentityError { Description = "角色不存在" });

        // 检查是否已是该角色的成员
        if (await _userManager.IsInRoleAsync(user, role.Name!))
        {
            return IdentityResult.Failed(new IdentityError
            {
                Code = "AlreadyInRole",
                Description = $"用户已经是 '{role.Name}' 角色的成员"
            });
        }

        var result = await _userManager.AddToRoleAsync(user, role.Name!);

        if (result.Succeeded)
        {
            _logger.LogInformation("将用户 {UserId} 添加到角色 {RoleName}",
                userId, role.Name);
        }

        return result;
    }

    /// <summary>
    /// 从角色中移除用户
    /// </summary>
    public async Task<IdentityResult> RemoveUserFromRoleAsync(string userId, string roleId)
    {
        var user = await _userManager.FindByIdAsync(userId);
        if (user == null)
            return IdentityResult.Failed(new IdentityError { Description = "用户不存在" });

        var role = await _roleManager.FindByIdAsync(roleId);
        if (role == null)
            return IdentityResult.Failed(new IdentityError { Description = "角色不存在" });

        // 安全保护：防止移除最后一个管理员
        if (string.Equals(role.Name, "Admin", StringComparison.OrdinalIgnoreCase))
        {
            var adminUsers = await _userManager.GetUsersInRoleAsync("Admin");
            if (adminUsers.Count <= 1 && adminUsers.Any(u => u.Id == userId))
            {
                return IdentityResult.Failed(new IdentityError
                {
                    Code = "CannotRemoveLastAdmin",
                    Description = "不能移除系统中唯一的管理员"
                });
            }
        }

        var result = await _userManager.RemoveFromRoleAsync(user, role.Name!);

        if (result.Succeeded)
        {
            _logger.LogInformation("从角色 {RoleName} 中移除用户 {UserId}",
                role.Name, userId);
        }

        return result;
    }

    /// <summary>
    /// 检查用户是否在指定角色中
    /// </summary>
    public async Task<bool> IsUserInRoleAsync(string userId, string roleName)
    {
        var user = await _userManager.FindByIdAsync(userId);
        if (user == null) return false;

        return await _userManager.IsInRoleAsync(user, roleName);
    }
}
```

### 3.2 DTO 定义

```csharp
// ====== 请求 DTO ======

public class CreateRoleRequest
{
    [Required(ErrorMessage = "请输入角色名称")]
    [StringLength(50, MinimumLength = 2)]
    [RegularExpression(@"^[a-zA-Z\u4e00-\u9fa5][a-zA-Z0-9_\-\u4e00-\u9fa5]*$",
        ErrorMessage = "角色名称只能包含字母、数字、中文、下划线和连字符")]
    public string Name { get; set; } = string.Empty;

    [MaxLength(200)]
    public string? Description { get; set; }
}

public class UpdateRoleRequest
{
    [Required]
    [StringLength(50, MinimumLength = 2)]
    public string Name { get; set; } = string.Empty;

    [MaxLength(200)]
    public string? Description { get; set; }

    public int SortOrder { get; set; } = 0;
}

// ====== 响应 DTO ======

public class RoleDto
{
    public string Id { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public bool IsSystemRole { get; set; }
    public int MemberCount { get; set; }
}

public class RoleDetailDto : RoleDto
{
    public int SortOrder { get; set; }
    public DateTime CreatedAt { get; set; }
    public List<UserSummaryDto> Members { get; set; } = new();
    public List<ClaimItemDto> Claims { get; set; } = new();
}

public class UserSummaryDto
{
    public string Id { get; set; } = string.Empty;
    public string UserName { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? Nickname { get; set; }
    public string? AvatarUrl { get; set; }
}

public class ClaimItemDto
{
    public string Type { get; set; } = string.Empty;
    public string Value { get; set; } = string.Empty;
}
```

---

## 四、在不同场景中使用角色授权

### 4.1 MVC Controller 中的使用

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers;

/// <summary>
/// 博客管理控制器 - 演示各种角色授权的使用
/// </summary>
public class BlogController : Controller
{
    private readonly IBlogService _blogService;
    private readonly IAuthorizationService _authService;

    public BlogController(IBlogService blogService, IAuthorizationService authService)
    {
        _blogService = blogService;
        _authService = authService;
    }

    // ==================== 公开页面 ====================

    // 所有人都可以访问（覆盖类级别授权）
    [AllowAnonymous]
    public async Task<IActionResult> Index(int page = 1)
    {
        var articles = await _blogService.GetPublishedArticlesAsync(page);
        return View(articles);
    }

    [AllowAnonymous]
    public async Task<IActionResult> Details(int id)
    {
        var article = await _blogService.GetByIdAsync(id);
        if (article == null) return NotFound();

        // 根据角色显示不同内容
        var canEdit = (await _authService.AuthorizeAsync(User, article, "CanEditArticle")).Succeeded;

        ViewBag.CanEdit = canEdit;
        return View(article);
    }

    // ==================== 需要登录的页面 ====================

    // 所有已登录用户都可以访问
    [Authorize]
    public async Task<IActionResult> MyArticles()
    {
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        var articles = await _blogService.GetUserArticlesAsync(userId!);
        return View(articles);
    }

    // ==================== 编辑者及以上权限 ====================

    // Editor 或 Admin 可以发布文章
    [Authorize(Roles = "Editor,Admin")]
    [HttpGet]
    public IActionResult Create()
    {
        return View();
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    [Authorize(Roles = "Editor,Admin")]
    public async Task<IActionResult> Create(CreateArticleViewModel model)
    {
        if (!ModelState.IsValid) return View(model);

        var authorId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        var authorName = User.FindFirstValue(ClaimTypes.Name);

        await _blogService.CreateAsync(new ArticleCreateModel
        {
            Title = model.Title,
            Content = model.Content,
            CategoryId = model.CategoryId,
            AuthorId = authorId!,
            AuthorName = authorName!
        });

        TempData["SuccessMessage"] = "文章发布成功";
        return RedirectToAction(nameof(Index));
    }

    // ==================== 仅管理员权限 ====================

    // 只有管理员可以进入后台管理
    [Authorize(Roles = "Admin")]
    public IActionResult AdminDashboard()
    {
        return View();
    }

    // 只有管理员可以删除文章
    [HttpPost]
    [ValidateAntiForgeryToken]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Delete(int id)
    {
        var article = await _blogService.GetByIdAsync(id);
        if (article == null) return NotFound();

        await _blogService.DeleteAsync(id);

        _logger.LogWarning("管理员 {AdminId} 删除了文章 {ArticleId}",
            User.FindFirstValue(ClaimTypes.NameIdentifier), id);

        TempData["SuccessMessage"] = "文章已删除";
        return RedirectToAction(nameof(Index));
    }

    // ==================== 复杂权限：资源所有者或管理员 ====================
    // 只能编辑自己的文章，但管理员可以编辑任意文章
    [HttpGet]
    [Authorize]
    public async Task<IActionResult> Edit(int id)
    {
        var article = await _blogService.GetByIdAsync(id);
        if (article == null) return NotFound();

        var currentUserId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        var isAdmin = User.IsInRole("Admin");

        // 不是作者也不是管理员 → 无权操作
        if (article.AuthorId != currentUserId && !isAdmin)
        {
            return Forbid();  // 返回 403
        }

        var model = new EditArticleViewModel
        {
            Id = article.Id,
            Title = article.Title,
            Content = article.Content,
            CategoryId = article.CategoryId
        };

        return View(model);
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    [Authorize]
    public async Task<IActionResult> Edit(EditArticleViewModel model)
    {
        if (!ModelState.IsValid) return View(model);

        var article = await _blogService.GetByIdAsync(model.Id);
        if (article == null) return NotFound();

        var currentUserId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        var isAdmin = User.IsInRole("Admin");

        if (article.AuthorId != currentUserId && !isAdmin)
        {
            return Forbid();
        }

        await _blogService.UpdateAsync(model.Id, model.Title, model.Content, model.CategoryId);
        TempData["SuccessMessage"] = "文章更新成功";

        return RedirectToAction(nameof(Details), new { id = model.Id });
    }
}
```

### 4.2 Razor Page 中的使用

```razor
@page
@model ManageModel
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

@* 页面级别的角色授权 *@
@attribute [Authorize(Roles = "Admin")]

<div class="container">
    <h2>系统管理</h2>

    @* 条件渲染：根据角色显示不同内容 *@
    @{
        var isAdmin = User.IsInRole("Admin");
        var isEditor = User.IsInRole("Editor");
    }

    @if (isAdmin)
    {
        <div class="alert alert-info">
            <strong>管理员面板：</strong> 您拥有完全的管理权限。
        </div>

        <!-- 管理员专属功能 -->
        <div class="card mb-3">
            <div class="card-header">用户管理</div>
            <div class="card-body">
                <a asp-page="/Admin/Users" class="btn btn-primary">
                    管理用户
                </a>
                <a asp-page="/Admin/Roles" class="btn btn-secondary">
                    管理角色
                </a>
            </div>
        </div>
    }

    @if (isEditor || isAdmin)
    {
        <div class="card mb-3">
            <div class="card-header">内容管理</div>
            <div class="card-body">
                <a asp-page="/Articles/Create" class="btn btn-success">
                    发布文章
                </a>
                <a asp-page="/Articles/PendingReview" class="btn btn-warning">
                    待审核文章 (@pendingCount)
                </a>
            </div>
        </div>
    }

    @* 使用 IAuthorizationService 进行程序化判断 *@
    @{
        var authResult = await AuthorizationService.AuthorizeAsync(User, "CanViewReports");
    }

    @if (authResult.Succeeded)
    {
        <div class="card">
            <div class="card-header">数据报表</div>
            <div class="card-body">
                <canvas id="reportsChart"></canvas>
            </div>
        </div>
    }
</div>
```

### 4.3 Web API 中的使用

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]  // 类级别：所有接口都需要认证
public class ArticlesApiController : ControllerBase
{
    private readonly IArticleService _articleService;

    public ArticlesApiController(IArticleService articleService)
    {
        _articleService = articleService;
    }

    /// <summary>
    /// 获取公开的文章列表（无需特殊角色）
    /// </summary>
    [HttpGet]
    [AllowAnonymous]
    public async Task<ActionResult<PagedResult<ArticleListDto>>> GetPublicArticles(
        [FromQuery] int page = 1, [FromQuery] int pageSize = 10)
    {
        var result = await _articleService.GetPublishedPagedAsync(page, pageSize);
        return Ok(result);
    }

    /// <summary>
    /// 创建文章（Editor 或 Admin）
    /// </summary>
    [HttpPost]
    [Authorize(Roles = "Editor,Admin")]
    public async Task<ActionResult<ArticleDto>> CreateArticle([FromBody] CreateArticleApiRequest request)
    {
        var authorId = User.FindFirstValue(JwtRegisteredClaimNames.Sub);
        var authorName = User.FindFirstValue(JwtRegisteredClaimNames.Name);

        var article = await _articleService.CreateAsync(new ArticleCreateModel
        {
            Title = request.Title,
            Content = request.Content,
            AuthorId = authorId!,
            AuthorName = authorName!
        });

        return CreatedAtAction(nameof(GetArticle), new { id = article.Id }, article);
    }

    /// <summary>
    /// 删除文章（仅 Admin）
    /// </summary>
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> DeleteArticle(int id)
    {
        var exists = await _articleService.ExistsAsync(id);
        if (!exists) return NotFound();

        await _articleService.DeleteAsync(id);

        return NoContent();
    }

    /// <summary>
    /// 更新文章（作者本人或 Admin）
    /// </summary>
    [HttpPut("{id}")]
    [Authorize]
    public async Task<IActionResult> UpdateArticle(int id, [FromBody] UpdateArticleApiRequest request)
    {
        var article = await _articleService.GetByIdAsync(id);
        if (article == null) return NotFound();

        var currentUserId = User.FindFirstValue(JwtRegisteredClaimNames.Sub);
        var isAdmin = User.IsInRole("Admin");

        // 授权检查：作者本人 或 管理员
        if (article.AuthorId != currentUserId && !isAdmin)
        {
            return StatusCode(StatusCodes.Status403Forbidden, new
            {
                code = 403,
                message = "您没有权限修改此文章（只能修改自己创建的文章）"
            });
        }

        await _articleService.UpdateAsync(id, request.Title, request.Content);
        return NoContent();
    }
}
```

---

## 五、完整 RBAC 权限管理系统示例

### 5.1 系统设计

```
三级权限体系设计:

┌────────────────────────────────────────────────────────────┐
│                      权限矩阵                               │
│                                                            │
│  功能模块        │ Admin │ Editor │ User │ Guest           │
│  ───────────────┼───────┼────────┼──────┼────────        │
│  浏览文章        │   ✓   │   ✓    │  ✓   │   ✓            │
│  发表评论        │   ✓   │   ✓    │  ✓   │               │
│  发布文章        │   ✓   │   ✓    │      │               │
│  编辑他人文章    │   ✓   │        │      │               │
│  编辑自己文章    │   ✓   │   ✓    │  ✓   │               │
│  删除文章        │   ✓   │        │      │               │
│  用户管理        │   ✓   │        │      │               │
│  角色管理        │   ✓   │        │      │               │
│  系统设置        │   ✓   │        │      │               │
│  数据统计        │   ✓   │   ✓    │      │               │
│                                                            │
│  注: ✓ 表示有权限，空白表示无权限                           │
└────────────────────────────────────────────────────────────┘
```

### 5.2 Program.cs 中的策略配置

```csharp
// 注册授权服务和策略
builder.Services.AddAuthorization(options =>
{
    // ====== 基于角色的基础策略 ======
    options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
    options.AddPolicy("EditorOrAbove", policy => policy.RequireRole("Admin", "Editor"));
    options.AddPolicy("AuthenticatedUser", policy => policy.RequireAuthenticatedUser());

    // ====== 组合策略（角色 + 其他条件）=====
    options.AddPolicy("TechDeptManager", policy =>
        policy.RequireRole("Manager")
              .RequireClaim("Department", "技术部"));

    // ====== 功能级策略 ======
    options.AddPolicy("CanPublishArticle", policy =>
        policy.RequireRole("Admin", "Editor"));

    options.AddPolicy("CanDeleteAnyArticle", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("CanManageUsers", policy =>
        policy.RequireRole("Admin"));
});
```

### 5.3 权限管理 API 控制器

```csharp
/// <summary>
/// 权限管理 API - 仅管理员可用
/// </summary>
[ApiController]
[Route("api/admin")]
[Authorize(Roles = "Admin")]
public class PermissionController : ControllerBase
{
    private readonly IRoleService _roleService;
    private readonly IUserService _userService;
    private readonly ILogger<PermissionController> _logger;

    public PermissionController(
        IRoleService roleService,
        IUserService userService,
        ILogger<PermissionController> logger)
    {
        _roleService = roleService;
        _userService = userService;
        _logger = logger;
    }

    #region === 角色管理 ===

    /// <summary>
    /// 获取所有角色及权限说明
    /// </summary>
    [HttpGet("roles")]
    public async Task<ActionResult<IEnumerable<RoleWithPermissionsDto>>> GetAllRolesWithPermissions()
    {
        var roles = await _roleService.GetAllRolesAsync();

        var result = roles.Select(r => new RoleWithPermissionsDto
        {
            Id = r.Id,
            Name = r.Name,
            Description = r.Description,
            IsSystemRole = r.IsSystemRole,
            MemberCount = r.MemberCount,
            Permissions = GetPermissionsForRole(r.Name)
        }).ToList();

        return Ok(result);
    }

    /// <summary>
    /// 获取某个角色的详细权限配置
    /// </summary>
    [HttpGet("roles/{roleId}/permissions")]
    public async Task<ActionResult<PermissionMatrixDto>> GetRolePermissions(string roleId)
    {
        var role = await _roleService.GetRoleByIdAsync(roleId);
        if (role == null) return NotFound();

        return Ok(new PermissionMatrixDto
        {
            RoleId = role.Id,
            RoleName = role.Name,
            Permissions = GetDetailedPermissionsForRole(role.Name)
        });
    }

    #endregion

    #region === 用户角色分配 ===

    /// <summary>
    /// 获取用户的角色列表
    /// </summary>
    [HttpGet("users/{userId}/roles")]
    public async Task<ActionResult<List<string>>> GetUserRoles(string userId)
    {
        var user = await _userService.GetUserByIdAsync(userId);
        if (user == null) return NotFound();

        var roles = await _userService.GetUserRolesAsync(userId);
        return Ok(roles);
    }

    /// <summary>
    /// 设置用户的角色（批量替换）
    /// </summary>
    [HttpPut("users/{userId}/roles")]
    public async Task<IActionResult> SetUserRoles(string userId, [FromBody] SetRolesRequest request)
    {
        var user = await _userService.GetUserByIdAsync(userId);
        if (user == null) return NotFound();

        // 不能修改自己的角色（安全保护）
        if (userId == User.FindFirstValue(ClaimTypes.NameIdentifier))
        {
            return BadRequest(new { message = "不能修改自己的角色" });
        }

        // 获取当前角色
        var currentRoles = await _userService.GetUserRolesAsync(userId);

        // 计算需要添加和移除的角色
        var toAdd = request.RoleIds.Except(currentRoles).ToList();
        var toRemove = currentRoles.Except(request.RoleIds).ToList();

        // 执行变更
        foreach (var roleName in toRemove)
        {
            await _roleService.RemoveUserFromRoleAsync(userId, roleName);
        }

        foreach (var roleId in toAdd)
        {
            var result = await _roleService.AddUserToRoleAsync(userId, roleId);
            if (!result.Succeeded)
            {
                return BadRequest(result.Errors);
            }
        }

        _logger.LogInformation("管理员 {AdminId} 修改了用户 {UserId} 的角色: 移除 [{Removed}]，添加 [{Added}]",
            User.FindFirstValue(ClaimTypes.NameIdentifier),
            userId,
            string.Join(", ", toRemove),
            string.Join(", ", toAdd));

        return Ok(new { message = "角色更新成功" });
    }

    #endregion

    #region === 辅助方法：权限定义 ===

    /// <summary>
    /// 获取角色对应的权限列表
    /// </summary>
    private static List<string> GetPermissionsForRole(string? roleName)
    {
        return roleName?.ToUpperInvariant() switch
        {
            "ADMIN" => new List<string>
            {
                "articles.view", "articles.create", "articles.edit_own",
                "articles.edit_others", "articles.delete",
                "comments.manage", "users.manage", "roles.manage",
                "settings.manage", "reports.view"
            },
            "EDITOR" => new List<string>
            {
                "articles.view", "articles.create", "articles.edit_own",
                "comments.manage", "reports.view"
            },
            "USER" => new List<string>
            {
                "articles.view", "articles.create", "articles.edit_own",
                "comments.create"
            },
            _ => new List<string> { "articles.view" }
        };
    }

    /// <summary>
    /// 获取详细的权限矩阵
    /// </summary>
    private static PermissionDetail[] GetDetailedPermissionsForRole(string? roleName)
    {
        var permissions = GetPermissionsForRole(roleName);
        return new[]
        {
            new PermissionDetail { Module = "文章浏览", Code = "articles.view", Allowed = permissions.Contains("articles.view") },
            new PermissionDetail { Module = "文章发布", Code = "articles.create", Allowed = permissions.Contains("articles.create") },
            new PermissionDetail { Module = "编辑自己的文章", Code = "articles.edit_own", Allowed = permissions.Contains("articles.edit_own") },
            new PermissionDetail { Module = "编辑他人的文章", Code = "articles.edit_others", Allowed = permissions.Contains("articles.edit_others") },
            new PermissionDetail { Module = "删除文章", Code = "articles.delete", Allowed = permissions.Contains("articles.delete") },
            new PermissionDetail { Module = "管理评论", Code = "comments.manage", Allowed = permissions.Contains("comments.manage") },
            new PermissionDetail { Module = "用户管理", Code = "users.manage", Allowed = permissions.Contains("users.manage") },
            new PermissionDetail { Module = "角色管理", Code = "roles.manage", Allowed = permissions.Contains("roles.manage") },
            new PermissionDetail { Module = "系统设置", Code = "settings.manage", Allowed = permissions.Contains("settings.manage") },
            new PermissionDetail { Module = "查看报表", Code = "reports.view", Allowed = permissions.Contains("reports.view") }
        };
    }

    #endregion
}

// ====== API DTO ======

public class RoleWithPermissionsDto
{
    public string Id { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string? Description { get; set; }
    public bool IsSystemRole { get; set; }
    public int MemberCount { get; set; }
    public List<string> Permissions { get; set; } = new();
}

public class PermissionMatrixDto
{
    public string RoleId { get; set; } = string.Empty;
    public string RoleName { get; set; } = string.Empty;
    public PermissionDetail[] Permissions { get; set; } = Array.Empty<PermissionDetail>();
}

public class PermissionDetail
{
    public string Module { get; set; } = string.Empty;
    public string Code { get; set; } = string.Empty;
    public bool Allowed { get; set; }
}

public class SetRolesRequest
{
    public List<string> RoleIds { get; set; } = new();
}
```

---

## 六、角色继承问题与解决方案

### 6.1 问题：ASP.NET Core 原生不支持角色继承

```
期望的层级关系:
Admin > Editor > User > Guest

如果 Admin 继承 Editor 的所有权限，
那么给 Admin 分配 Editor 角色就能获得 Editor 的权限？

❌ ASP.NET Core 的 [Authorize(Roles)] 不支持这种语义！
[Authorize(Roles = "Admin")] ≠ [Authorize(Roles = "Admin,Editor")]
```

### 6.2 解决方案一：多角色分配

```csharp
// 方案：给高级用户同时分配多个角色
// Admin 用户同时拥有: Admin + Editor + User 角色
// Editor 用户同时拥有: Editor + User 角色

// 这样 [Authorize(Roles = "Editor,Admin")] 就能正确匹配
```

### 6.3 解决方案二：使用 Policy 封装

```csharp
// 注册策略时明确包含所有子角色
options.AddPolicy("EditorOrAbove", policy =>
    policy.RequireRole("Admin", "Editor"));

options.AddPolicy("UserOrAbove", policy =>
    policy.RequireRole("Admin", "Editor", "User"));
```

### 6.4 解决方案三：数据库驱动的动态权限（推荐用于大型系统）

```csharp
/// <summary>
/// 从数据库读取角色-权限关系，实现动态权限检查
/// </summary>
public class DynamicRoleRequirement : IAuthorizationRequirement
{
    public string RequiredPermission { get; }

    public DynamicRoleRequirement(string requiredPermission)
    {
        RequiredPermission = requiredPermission;
    }
}

public class DynamicRoleHandler : AuthorizationHandler<DynamicRoleRequirement>
{
    private readonly IPermissionCache _permissionCache;

    public DynamicRoleHandler(IPermissionCache permissionCache)
    {
        _permissionCache = permissionCache;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        DynamicRoleRequirement requirement)
    {
        var userId = context.User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (userId == null) return;

        // 从缓存/数据库获取用户的权限列表
        var userPermissions = await _permissionCache.GetUserPermissionsAsync(userId);

        if (userPermissions.Contains(requirement.RequiredPermission))
        {
            context.Succeed(requirement);
        }
    }
}

// 注册
builder.Services.AddSingleton<IAuthorizationHandler, DynamicRoleHandler>();
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanDeleteArticle",
        policy => policy.AddRequirements(new DynamicRoleRequirement("articles.delete")));
});
```

---

## 七、安全最佳实践清单

### DO ✅

- **DO** 使用 `RequireAuthenticatedUser()` 作为默认的全局策略
- **DO** 将角色名定义为常量，避免魔法字符串散落在各处
- **DO** 敏感操作（删除、批量操作）始终做二次确认
- **DO** 记录所有角色变更操作的审计日志
- **DO** 保护好最后一个管理员账户不被误删/降权
- **DO** 使用 `[AutoValidateAntiforgeryToken]` 全局启用 CSRF 防护
- **DO** 定期审查角色分配情况，清理不必要的权限
- **DO** API 接口返回标准化的错误格式（401/403）

### DON'T ❌

- **DON'T** 不要在前端隐藏按钮来代替服务端权限验证（前端只是体验优化！）
- **DON'T** 不要信任客户端传来的角色信息（必须由服务端签发）
- **DON'T** 不要用 `string.Equals` 手动比较角色名（应使用 `IsInRole` 或框架机制）
- **DON'T** 不要忽略 `Forbid()` 和 `Unauthorized()` 的区别
- **DON'T** 不要在循环中进行数据库查询获取角色信息（使用缓存）
- **DON'T** 不要赋予新注册用户超过必要的默认角色
- **DON'T** 不要忘记在 Razor Pages 中使用 `@attribute [Authorize]`
- **DON'T** 不要把业务逻辑写在 Attribute 里（保持关注点分离）

---

## 八、练习题

### 练习 1：角色设计
为一个在线教育平台设计 RBAC 角色体系，至少包含以下角色：
- SuperAdmin（平台超管）
- SchoolAdmin（学校管理员）
- Teacher（教师）
- Student（学生）
请画出每个角色的权限矩阵表。

### 练习 2：三种方式实战
对同一个功能（如"导出学生名单"），分别用三种方式实现角色授权：
1. 声明式：`[Authorize(Roles = "...")]`
2. 策略式：`[Authorize(Policy = "...")]`
3. 命令式：`_authService.AuthorizeAsync(...)`
比较三种方式的优劣。

### 练习 3：资源所有权
实现一个需求："用户只能删除自己创建的评论，但管理员可以删除任意评论"。要求：
- 在 Controller 中实现完整的 Action 方法
- 返回正确的 HTTP 状态码（200/403/404）
- 包含适当的日志记录

### 练习 4：角色继承模拟
虽然 ASP.NET Core 不原生支持角色继承，请设计一种方案来模拟以下层级关系：
`SuperAdmin > Admin > Moderator > User`
使得 SuperAdmin 能自动满足所有低级角色的授权检查。

### 练习 5：权限审计
编写一个管理页面，展示以下信息：
- 每个角色的成员列表和人数
- 最近 7 天内的角色变更记录
- 异常情况提醒（如某用户同时拥有过多角色、某角色无人使用等）

---

## 九、延伸阅读

- [Microsoft Docs: ASP.NET Core 中基于角色的授权](https://docs.microsoft.com/zh-cn/aspnet/core/security/authorization/roles)
- [NIST RBAC 标准](https://csrc.nist.gov/pubs/sp800/162/final)
- [OWASP: Access Control Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html)
- [Microsoft Docs: 自定义基于策略的授权](https://docs.microsoft.com/zh-cn/asp.net/core/security/authorization/policies)

---

> **下一节预告**：我们将学习 **基于策略的授权 (Policy)**，了解如何通过 Requirement、Handler、Policy 三要素实现比简单角色更灵活的细粒度权限控制。
