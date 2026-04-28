# 内存缓存 (IMemoryCache) 完全指南

> **学习目标**：掌握 ASP.NET Core 中内存缓存的完整使用方法，包括基本API、高级配置、实际场景应用和最佳实践
>
> **难度等级**：⭐⭐⭐ 中级
>
> **前置知识**：依赖注入基础、异步编程基础
>
> **预计时间**：45分钟

---

## 1. 为什么需要缓存？

### 1.1 现实中的性能瓶颈

想象一下这个场景：

**[场景截图：用户请求流程图]**
```
用户请求 → Controller → Service → 数据库查询 → 返回结果
                    ↑
              每次都查数据库 = 慢！
```

**问题所在**：
- 数据库查询是I/O密集型操作，通常需要10-100ms
- 高并发时，数据库成为系统瓶颈
- 相同的数据被反复查询（如网站配置、用户信息、热门文章）

**缓存带来的提升**：
- 内存读取速度：~100纳秒（比数据库快100万倍！）
- 减少数据库负载，提升整体吞吐量
- 用户响应速度显著提升

### 1.2 生活类比

**缓存就像便利贴/备忘录**：

| 场景 | 无缓存 | 有缓存 |
|------|--------|--------|
| 记电话号码 | 每次都翻通讯录 | 写在便利贴上，直接看 |
| 记公式 | 每次都查书 | 记在脑子里，直接用 |
| 记用户信息 | 每次都查数据库 | 存在内存里，直接取 |

---

## 2. 缓存的基本原理

### 2.1 核心思想：空间换时间

```
传统方式：时间换空间
┌─────────────────────────────────────┐
│  内存：小（省空间）                   │
│  数据库：大但慢                       │
│  每次都从数据库读 = 慢但省内存         │
└─────────────────────────────────────┘

缓存方式：空间换时间
┌─────────────────────────────────────┐
│  内存：大（占用更多内存）              │
│  数据库：作为数据源                   │
│  常用数据放内存 = 快但占内存           │
└─────────────────────────────────────┘
```

### 2.2 缓存的工作流程

**[流程图：缓存命中 vs 缓存未命中]**

```csharp
// 伪代码展示缓存逻辑
public async Task<User> GetUserAsync(int userId)
{
    // 1. 先尝试从缓存获取
    var cachedUser = _cache.Get<User>($"user:{userId}");

    if (cachedUser != null)
    {
        // ✅ 缓存命中 - 直接返回（快！）
        return cachedUser;
    }

    // 2. 缓存未命中 - 从数据库查询
    var user = await _dbContext.Users.FindAsync(userId);

    // 3. 存入缓存，下次就不用查数据库了
    _cache.Set($"user:{userId}", user, TimeSpan.FromMinutes(30));

    return user;
}
```

---

## 3. IMemoryCache 接口详解

### 3.1 接口定义

`IMemoryCache` 是 .NET 提供的内存缓存接口，位于 `Microsoft.Extensions.Caching.Memory` 命名空间。

**核心接口成员**：

```csharp
public interface IMemoryCache : IDisposable
{
    // 尝试获取缓存项
    bool TryGetValue(object key, out object? value);

    // 创建或获取缓存项（最常用！）
    TItem? GetOrCreate<TItem>(object key, Func<ICacheEntry, TItem> factory);
    Task<TItem?> GetOrCreateAsync<TItem>(object key, Func<ICacheEntry, Task<TItem>> factory);

    // 获取缓存项
    TItem? Get<TItem>(object key);

    // 设置缓存项
    void Set<TItem>(object key, TItem value, MemoryCacheEntryOptions? options = null);

    // 移除缓存项
    void Remove(object key);
}
```

### 3.2 注册缓存服务

在 `Program.cs` 中注册内存缓存服务：

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Caching.Memory;

var builder = WebApplication.CreateBuilder(args);

// ✅ 注册内存缓存服务（必须这一步！）
builder.Services.AddMemoryCache();

// 其他服务...
builder.Services.AddControllers();
var app = builder.Build();

app.MapControllers();
app.Run();
```

**验证点**：注册后可以在任何地方通过构造函数注入 `IMemoryCache`

### 3.3 注入和使用 IMemoryCache

**示例：在 Service 中使用缓存**

```csharp
using Microsoft.Extensions.Caching.Memory;

public class UserService
{
    private readonly IMemoryCache _cache;
    private readonly ApplicationDbContext _dbContext;

    public UserService(IMemoryCache cache, ApplicationDbContext dbContext)
    {
        _cache = cache;       // ✅ 通过构造函数注入
        _dbContext = dbContext;
    }

    public async Task<User?> GetUserByIdAsync(int userId)
    {
        string cacheKey = $"user:{userId}";

        // 使用 GetOrCreateAsync - 最推荐的写法
        return await _cache.GetOrCreateAsync(cacheKey, async entry =>
        {
            // ⚠️ 这个委托只在缓存未命中时执行
            Console.WriteLine("从数据库查询用户...");

            var user = await _dbContext.Users.FindAsync(userId);

            if (user != null)
            {
                // 设置缓存选项
                entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
                entry.Priority = CacheItemPriority.Normal;
            }

            return user;
        });
    }
}
```

**[截图：控制台输出 - 第一次调用显示"从数据库查询"，第二次调用不显示]**

---

## 4. 基本API详解

### 4.1 GetOrCreateAsync() - 获取或创建（⭐ 最常用）

这是最推荐的方法，原子性地完成"检查缓存→若没有则创建"的操作。

```csharp
// 完整示例：获取热门文章列表
public async Task<List<BlogPost>> GetTopPostsAsync()
{
    return await _cache.GetOrCreateAsync("top_posts:10", async entry =>
    {
        // 设置缓存30分钟后过期
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);

        // 设置优先级
        entry.Priority = CacheItemPriority.High;

        // 设置大小（用于限制缓存总大小）
        entry.Size = 1;

        Console.WriteLine("[缓存未命中] 正在从数据库查询热门文章...");

        // 从数据库查询
        return await _dbContext.BlogPosts
            .Where(p => p.IsPublished)
            .OrderByDescending(p => p.ViewCount)
            .Take(10)
            .ToListAsync();
    });
}
```

**使用示例**：

```csharp
// 第一次调用 - 会执行数据库查询并缓存
var posts1 = await _userService.GetTopPostsAsync();  // 控制台输出: [缓存未命中]...

// 第二次调用 - 直接从缓存返回，不查数据库
var posts2 = await _userService.GetTopPostsAsync();  // 控制台无输出

// 30分钟后再次调用 - 缓存过期，重新查询
var posts3 = await _userService.GetTopPostsAsync();  // 控制台输出: [缓存未命中]...
```

### 4.2 Get() / Set() - 分别获取和设置

适用于需要更精细控制的场景。

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;

    public ProductService(IMemoryCache _cache)
    {
        this._cache = _cache;
    }

    // 获取产品详情
    public Product? GetProduct(int productId)
    {
        string cacheKey = $"product:{productId}";

        // 方式1：使用Get方法
        var product = _cache.Get<Product>(cacheKey);

        if (product == null)
        {
            // 缓存不存在，从数据库查询
            product = _dbContext.Products.Find(productId);

            if (product != null)
            {
                // 手动设置到缓存
                _cache.Set(cacheKey, product, new MemoryCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1),
                    Priority = CacheItemPriority.Normal
                });
            }
        }

        return product;
    }
}
```

### 4.3 Remove() - 手动移除缓存

当数据发生变化时，需要手动清除旧缓存。

```csharp
public class UserService
{
    // 更新用户信息后，清除该用户的缓存
    public async Task<User> UpdateUserAsync(int userId, UpdateUserRequest request)
    {
        var user = await _dbContext.Users.FindAsync(userId)
            ?? throw new NotFoundException("用户不存在");

        // 更新用户信息
        user.Nickname = request.Nickname;
        user.Bio = request.Bio;
        await _dbContext.SaveChangesAsync();

        // ✅ 清除缓存，确保下次获取的是最新数据
        _cache.Remove($"user:{userId}");

        return user;
    }

    // 批量清除相关缓存
    public void ClearUserRelatedCache(int userId)
    {
        _cache.Remove($"user:{userId}");
        _cache.Remove($"user:{userId}:profile");
        _cache.Remove($"user:{userId}:posts");
        // 或者使用前缀匹配（需要自定义实现）
    }
}
```

---

## 5. 缓存选项 (MemoryCacheEntryOptions) 详解

`MemoryCacheEntryOptions` 是缓存的灵魂，它决定了缓存的行为。

### 5.1 AbsoluteExpiration - 绝对过期时间

**含义**：无论是否被访问，到了指定时间就一定清除。

```csharp
var options = new MemoryCacheEntryOptions
{
    // 方式1：相对于现在的时间
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30),  // 30分钟后过期

    // 方式2：绝对的UTC时间点
    AbsoluteExpiration = DateTimeOffset.UtcNow.AddHours(2)  // 在某个具体时间点过期
};
```

**适用场景**：
- 配置信息缓存（每小时刷新一次）
- 新闻列表（每5分钟更新）
- 汇率数据（定时更新）

**[示意图：绝对过期时间轴]**
```
时间轴：───[设置缓存]─────────────[30分钟后自动删除]───
                      ↑
               不管有没有人访问，到期就删
```

### 5.2 SlidingExpiration - 滑动过期时间

**含义**：如果持续访问，就一直续期；闲置超过指定时间才清除。

```csharp
var options = new MemoryCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(10)  // 闲置10分钟就过期
};
```

**适用场景**：
- 用户会话数据（活跃用户保持登录）
- 购物车数据（有操作就保留）
- 热门商品信息（有人看就保留）

**[示意图：滑动过期时间轴]**
```
时间轴：───[设置缓存]─[访问+续期]─[访问+续期]─[闲置10分钟]─[过期]
                  ↑            ↑                     ↑
             重置计时器   重置计时器            超过10分钟没访问
```

**⚠️ 重要提示**：建议配合 `AbsoluteExpiration` 使用，防止"永久不过期"。

```csharp
// ✅ 最佳实践：双重过期策略
var options = new MemoryCacheEntryOptions
{
    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(2),  // 最多存活2小时
    SlidingExpiration = TimeSpan.FromMinutes(10)              // 闲置10分钟过期
};
```

### 5.3 Priority - 缓存优先级

当内存不足时，系统会根据优先级淘汰缓存条目。

```csharp
var options = new MemoryCacheEntryOptions
{
    Priority = CacheItemPriority.High  // 优先级设置
};
```

**优先级级别**：

| 枚举值 | 说明 | 适用场景 |
|--------|------|----------|
| `Low` | 低优先级，最先被淘汰 | 临时计算结果、可重新生成的数据 |
| `Normal` | 正常优先级（默认） | 一般的缓存数据 |
| `High` | 高优先级，最后被淘汰 | 重要配置、认证Token |
| `NeverRemove` | 永不移除（除非手动Remove或过期） | 关键业务数据 |

**示例**：

```csharp
// 系统配置 - 高优先级，尽量保留
_cache.Set("system_config", config, new MemoryCacheEntryOptions
{
    Priority = CacheItemPriority.NeverRemove,
    AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(1)
});

// 临时统计 - 低优先级，内存紧张时可丢弃
_cache.Set("temp_stats", stats, new MemoryCacheEntryOptions
{
    Priority = CacheItemPriority.Low,
    SlidingExpiration = TimeSpan.FromMinutes(5)
});
```

### 5.4 Size - 缓存条目大小限制

可以限制缓存的总大小，防止内存溢出。

```csharp
// Program.cs 中配置缓存大小限制
builder.Services.AddMemoryCache(options =>
{
    options.SizeLimit = 1024 * 1024 * 100;  // 限制总大小为100MB
});

// 设置每个条目的大小
_cache.Set("large_data", bigData, new MemoryCacheEntryOptions
{
    Size = 1024  // 这个条目占用1KB的大小配额
});
```

### 5.5 PostEvictionCallbacks - 缓存被移除时的回调

当缓存项被移除时（过期、手动删除、内存不足淘汰），可以执行回调逻辑。

```csharp
var options = new MemoryCacheEntryOptions()
    .RegisterPostEvictionCallback((key, value, reason, state) =>
    {
        Console.WriteLine($"缓存项 {key} 被移除，原因：{reason}");
        Console.WriteLine($"移除时的值：{value}");

        // 可以在这里记录日志、发送通知等
        if (reason == EvictionReason.Expired)
        {
            Console.WriteLine("原因是：已过期");
        }
        else if (reason == EvictionReason.Removed)
        {
            Console.WriteLine("原因是：手动移除");
        }
        else if (reason == EvictionReason.Capacity)
        {
            Console.WriteLine("原因是：内存不足被淘汰");
        }
    });

_cache.Set("my_key", "my_value", options);
```

**EvictionReason 枚举**：
- `None` - 未指定原因
- `Removed` - 手动调用 Remove()
- `Replaced` - 同一个key被重新Set
- `Expired` - 过期（Absolute或Sliding）
- `TokenExpired` - 取消令牌触发
- `Capacity` - 内存不足被淘汰

---

## 6. 缓存键的设计规范

良好的键设计能让缓存系统更易维护。

### 6.1 命名规范

```csharp
// ✅ 好的命名方式：前缀 + 层次结构
public static class CacheKeys
{
    // 用户相关
    public const string UserPrefix = "user:";
    public static string User(int id) => $"user:{id}";
    public static string UserProfile(int id) => $"user:{id}:profile";
    public static string UserPermissions(int id) => $"user:{id}:permissions";

    // 文章相关
    public const string PostsPrefix = "posts:";
    public static string Post(int id) => $"post:{id}";
    public static string TopPosts(int count) => $"posts:top:{count}";
    public static string PostsByCategory(int categoryId) => $"posts:category:{categoryId}";

    // 配置相关
    public const string SystemConfig = "system:config";
    public const string SiteSettings = "system:settings";

    // 标签云
    public const string TagCloud = "tags:cloud";
}

// ❌ 不好的命名方式
_cache.Set("u1", user);              // 不清晰
_cache.Set("data", list);            // 太泛
_cache.Set("user_123_info", info);   // 格式不一致
```

### 6.2 键的设计原则

| 原则 | 说明 | 示例 |
|------|------|------|
| **可读性** | 键名要能表达缓存内容的含义 | `"user:123"` 比 `"u123"` 好 |
| **层次化** | 使用冒号分隔不同层级 | `"post:category:5:page:1"` |
| **一致性** | 整个项目使用统一的命名规范 | 全部小写 + 冒号分隔 |
| **唯一性** | 不同数据必须使用不同的键 | 避免冲突 |
| **可管理性** | 前缀设计便于批量清理 | `"user:*"` 可清理所有用户缓存 |

---

## 7. 缓存三大经典问题

### 7.1 缓存穿透 (Cache Penetration)

**问题描述**：查询一个不存在的数据，每次都穿透缓存直接查数据库。

**场景**：恶意用户不断查询不存在的 userId（如 userId=-1 或超大的ID）

```
恶意请求 → 查缓存(null) → 查数据库(null) → 返回null
                                    ↑
                            每次都打到数据库！
```

**解决方案**：

```csharp
// 方案1：缓存空值（设置短过期时间）
public async Task<User?> GetUserAsync(int userId)
{
    string cacheKey = $"user:{userId}";

    var cached = _cache.Get<User?>(cacheKey);
    if (cached != null)
    {
        // 即使是默认值对象，也说明之前查过
        return cached.Id > 0 ? cached : null;
    }

    var user = await _dbContext.Users.FindAsync(userId);

    // 无论是否存在，都放入缓存
    if (user != null)
    {
        _cache.Set(cacheKey, user, TimeSpan.FromMinutes(30));
    }
    else
    {
        // ✅ 缓存空值，防止穿透（短过期时间）
        _cache.Set(cacheKey, new User { Id = -1 }, TimeSpan.FromMinutes(5));
    }

    return user;
}

// 方案2：布隆过滤器（Bloom Filter）- 适用于大数据量
```

### 7.2 缓存击穿 (Cache Breakdown)

**问题描述**：热点Key突然过期，大量并发请求同时打到数据库。

**场景**：微博热搜第一名缓存过期，瞬间百万请求同时查询数据库

```
时间点T：缓存过期
     ↓
请求1 ──→ 查缓存(null) → 查数据库（正在查询...）
请求2 ──→ 查缓存(null) → 查数据库（正在查询...）
请求3 ──→ 查缓存(null) → 查数据库（正在查询...）
...                             ↑
                          数据库被打爆！
```

**解决方案**：

```csharp
// 方案1：互斥锁（只让一个请求去查数据库）
public async Task<T> GetWithLockAsync<T>(string key, Func<Task<T>> factory)
{
    var result = _cache.Get<T>(key);
    if (result != null)
        return result;

    // 使用SemaphoreSlim实现互斥
    var lockKey = $"lock:{key}";
    if (!_cache.TryGetValue(lockKey, out _))
    {
        // 获取锁成功，去查数据库
        _cache.Set(lockKey, true, TimeSpan.FromSeconds(10));

        try
        {
            result = await factory();
            _cache.Set(key, result, TimeSpan.FromMinutes(30));
        }
        finally
        {
            _cache.Remove(lockKey);
        }
    }
    else
    {
        // 没获取到锁，等待后重试
        await Task.Delay(100);
        return await GetWithLockAsync(key, factory);  // 递归重试
    }

    return result;
}

// 方案2：逻辑过期（不设置真正的过期，后台异步刷新）
```

### 7.3 缓存雪崩 (Cache Avalanche)

**问题描述**：大量缓存同时过期，导致数据库压力骤增。

**场景**：系统重启后，所有缓存同时失效；或者批量设置的缓存都是相同的过期时间

```
时间点T：大批量缓存同时过期
     ↓
请求洪流 ──→ 全部缓存未命中 ──→ 全部打向数据库
                                  ↑
                              数据库崩溃！
```

**解决方案**：

```csharp
// ✅ 解决方案：过期时间加随机值，避免同时过期
public static MemoryCacheEntryOptions CreateRandomExpiration()
{
    var random = new Random();
    var baseMinutes = 30;  // 基础30分钟
    var randomMinutes = random.Next(0, 10);  // 随机0-10分钟

    return new MemoryCacheEntryOptions
    {
        AbsoluteExpirationRelativeToNow =
            TimeSpan.FromMinutes(baseMinutes + randomMinutes),  // 30-40分钟之间随机
        Priority = CacheItemPriority.Normal
    };
}

// 使用
_cache.Set("key", value, CreateRandomExpiration());
```

---

## 8. 实际场景示例

### 8.1 场景一：用户信息缓存

**需求**：用户登录后缓存用户Profile，减少数据库查询

```csharp
public class UserProfileService
{
    private readonly IMemoryCache _cache;
    private readonly ApplicationDbContext _dbContext;
    private readonly ILogger<UserProfileService> _logger;

    public UserProfileService(
        IMemoryCache cache,
        ApplicationDbContext dbContext,
        ILogger<UserProfileService> logger)
    {
        _cache = cache;
        _dbContext = dbContext;
        _logger = logger;
    }

    /// <summary>
    /// 获取用户Profile（带缓存）
    /// </summary>
    public async Task<UserProfileResponse> GetUserProfileAsync(int userId)
    {
        string cacheKey = CacheKeys.UserProfile(userId);

        return await _cache.GetOrCreateAsync(cacheKey, async entry =>
        {
            _logger.LogInformation("缓存未命中，从数据库加载用户 Profile: {UserId}", userId);

            var user = await _dbContext.Users
                .Include(u => u.Role)
                .FirstOrDefaultAsync(u => u.Id == userId);

            if (user == null)
                throw new NotFoundException("用户不存在");

            var profile = new UserProfileResponse
            {
                Id = user.Id,
                Email = user.Email,
                Nickname = user.Nickname,
                Avatar = user.Avatar,
                Bio = user.Bio,
                RoleName = user.Role?.Name,
                CreatedAt = user.CreatedAt
            };

            // 缓存30分钟，闲置15分钟过期
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
            entry.SlidingExpiration = TimeSpan.FromMinutes(15);
            entry.Priority = CacheItemPriority.High;

            // 注册移除回调
            entry.RegisterPostEvictionCallback((key, value, reason, state) =>
            {
                _logger.LogInformation(
                    "用户缓存被移除 - Key: {Key}, Reason: {Reason}",
                    key, reason);
            });

            return profile;
        });
    }

    /// <summary>
    /// 更新用户资料（记得清除缓存！）
    /// </summary>
    public async Task<UserProfileResponse> UpdateProfileAsync(
        int userId, UpdateProfileRequest request)
    {
        var user = await _dbContext.Users.FindAsync(userId)
            ?? throw new NotFoundException("用户不存在");

        // 更新字段
        user.Nickname = request.Nickname;
        user.Bio = request.Bio;
        user.Avatar = request.Avatar;
        user.UpdatedAt = DateTime.UtcNow;

        await _dbContext.SaveChangesAsync();

        // ✅ 清除旧缓存，确保下次获取最新数据
        _cache.Remove(CacheKeys.UserProfile(userId));

        _logger.LogInformation("用户资料已更新并清除缓存: {UserId}", userId);

        return await GetUserProfileAsync(userId);  // 重新获取（会写入新缓存）
    }
}
```

**Controller层使用**：

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserProfileService _profileService;

    public UsersController(UserProfileService profileService)
    {
        _profileService = profileService;
    }

    [HttpGet("{userId}/profile")]
    public async Task<ActionResult<UserProfileResponse>> GetUserProfile(int userId)
    {
        var profile = await _profileService.GetUserProfileAsync(userId);
        return Ok(profile);
    }

    [HttpPut("{userId}/profile")]
    public async Task<ActionResult<UserProfileResponse>> UpdateProfile(
        int userId, [FromBody] UpdateProfileRequest request)
    {
        var profile = await _profileService.UpdateProfileAsync(userId, request);
        return Ok(profile);
    }
}
```

**验证步骤**：
1. 调用 GET /api/users/1/profile - 观察日志显示"缓存未命中"
2. 再次调用同一接口 - 日志不再显示（缓存命中）
3. 调用 PUT 更新资料
4. 再次GET - 日志又显示"缓存未命中"（缓存已被清除）

### 8.2 场景二：热门文章列表缓存

**需求**：首页展示Top10热门文章，缓存5分钟

```csharp
public class BlogPostService
{
    private readonly IMemoryCache _cache;
    private readonly ApplicationDbContext _dbContext;

    public BlogPostService(IMemoryCache cache, ApplicationDbContext dbContext)
    {
        _cache = cache;
        _dbContext = dbContext;
    }

    /// <summary>
    /// 获取首页热门文章（带分页）
    /// </summary>
    public async Task<PagedResult<BlogPostDto>> GetPopularPostsAsync(
        int page = 1, int pageSize = 10)
    {
        // 对于分页数据，可以使用组合键
        string cacheKey = $"{CacheKeys.TopPosts(pageSize)}:page:{page}";

        return await _cache.GetOrCreateAsync(cacheKey, async entry =>
        {
            Console.WriteLine($"[缓存未命中] 加载第{page}页热门文章...");

            var query = _dbContext.BlogPosts
                .Include(p => p.Author)
                .Include(p => p.Category)
                .Where(p => p.IsPublished)
                .OrderByDescending(p => p.ViewCount)
                .ThenByDescending(p => p.PublishedAt);

            var totalCount = await query.CountAsync();

            var items = await query
                .Skip((page - 1) * pageSize)
                .Take(pageSize)
                .Select(p => new BlogPostDto
                {
                    Id = p.Id,
                    Title = p.Title,
                    Summary = p.Summary,
                    CoverImage = p.CoverImage,
                    ViewCount = p.ViewCount,
                    AuthorName = p.Author.Nickname,
                    CategoryName = p.Category.Name,
                    PublishedAt = p.PublishedAt
                })
                .ToListAsync();

            // 热门文章缓存5分钟即可（变化频繁）
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            entry.Priority = CacheItemPriority.Normal;

            return new PagedResult<BlogPostDto>
            {
                Items = items,
                TotalCount = totalCount,
                Page = page,
                PageSize = pageSize
            };
        });
    }

    /// <summary>
    /// 文章浏览次数+1（注意：这里不要缓存，要实时更新）
    /// </summary>
    public async Task IncrementViewCountAsync(int postId)
    {
        await _dbContext.BlogPosts
            .Where(p => p.Id == postId)
            .ExecuteUpdateAsync(setters => setters
                .SetProperty(p => p.ViewCount, p => p.ViewCount + 1));

        // 浏览数更新后，可以考虑清除相关缓存
        _cache.Remove(CacheKeys.Post(postId));
    }
}
```

**Controller**:

```csharp
[HttpGet("popular")]
public async Task<ActionResult<PagedResult<BlogPostDto>>> GetPopularPosts(
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 10)
{
    var result = await _blogPostService.GetPopularPostsAsync(page, pageSize);
    return Ok(result);
}
```

### 8.3 场景三：配置信息缓存

**需求**：系统配置信息不常变，可以长时间缓存或永久缓存

```csharp
public class ConfigurationService
{
    private readonly IMemoryCache _cache;
    private readonly IConfiguration _configuration;

    public ConfigurationService(IMemoryCache cache, IConfiguration configuration)
    {
        _cache = cache;
        _configuration = configuration;
    }

    /// <summary>
    /// 获取站点配置（缓存1小时）
    /// </summary>
    public async Task<SiteConfig> GetSiteConfigAsync()
    {
        return await _cache.GetOrCreateAsync(CacheKeys.SiteSettings, async entry =>
        {
            // 配置信息缓存较长时间
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1);
            entry.Priority = CacheItemPriority.NeverRemove;

            return new SiteConfig
            {
                SiteName = _configuration["SiteSettings:Name"] ?? "我的博客",
                SiteDescription = _configuration["SiteSettings:Description"],
                PostsPerPage = int.Parse(_configuration["SiteSettings:PostsPerPage"] ?? "10"),
                EnableComments = bool.Parse(_configuration["SiteSettings:EnableComments"] ?? "true"),
                CommentModeration = bool.Parse(_configuration["SiteSettings:CommentModeration"] ?? "true")
            };
        });
    }

    /// <summary>
    /// 强制刷新配置缓存（管理员修改配置后调用）
    /// </summary>
    public void RefreshConfigCache()
    {
        _cache.Remove(CacheKeys.SiteSettings);
    }
}
```

---

## 9. 缓存的监控和统计

### 9.1 自定义缓存统计

```csharp
/// <summary>
/// 缓存监控服务 - 统计命中率等信息
/// </summary>
public class CacheMonitorService
{
    private long _hitCount = 0;
    private long _missCount = 0;
    private readonly object _lock = new object();

    public void RecordHit()
    {
        Interlocked.Increment(ref _hitCount);
    }

    public void RecordMiss()
    {
        Interlocked.Increment(ref _missCount);
    }

    public CacheStatistics GetStatistics()
    {
        var total = _hitCount + _missCount;
        return new CacheStatistics
        {
            HitCount = _hitCount,
            MissCount = _missCount,
            TotalRequests = total,
            HitRate = total > 0 ? (double)_hitCount / total : 0
        };
    }
}

public class CacheStatistics
{
    public long HitCount { get; set; }
    public long MissCount { get; set; }
    public long TotalRequests { get; set; }
    public double HitRate { get; set; }  // 命中率 0-1
}
```

### 9.2 监控端点

```csharp
[ApiController]
[Route("api/[controller]")]
public class HealthController : ControllerBase
{
    private readonly CacheMonitorService _cacheMonitor;

    public HealthController(CacheMonitorService cacheMonitor)
    {
        _cacheMonitor = cacheMonitor;
    }

    [HttpGet("cache-stats")]
    public ActionResult<CacheStatistics> GetCacheStats()
    {
        var stats = _cacheMonitor.GetStatistics();
        return Ok(new
        {
            stats.HitCount,
            stats.MissCount,
            stats.TotalRequests,
            HitRate = $"{stats.HitRate:P2}",  // 百分比格式
            Message = stats.HitRate >= 0.8 ? "缓存状态良好" : "缓存命中率偏低"
        });
    }
}
```

**响应示例**：
```json
{
  "hitCount": 9527,
  "missCount": 473,
  "totalRequests": 10000,
  "hitRate": "95.27%",
  "message": "缓存状态良好"
}
```

---

## 10. 注意事项与最佳实践

### 10.1 不要缓存敏感信息

```csharp
// ❌ 错误：缓存密码、Token等敏感信息
_cache.Set("user_password", user.PasswordHash);
_cache.Set("auth_token", jwtToken);

// ✅ 正确：只缓存非敏感的业务数据
_cache.Set("user_profile", userProfile);  // 不包含密码
```

### 10.2 合理设置过期时间

| 数据类型 | 推荐过期时间 | 原因 |
|----------|-------------|------|
| 用户Session | 15-30分钟 | 安全考虑 |
| 用户Profile | 30分钟-1小时 | 变化频率适中 |
| 热门文章列表 | 5-10分钟 | 内容变化频繁 |
| 系统配置 | 1-24小时 | 很少变化 |
| 字典数据（性别、国家等） | 1小时或更长 | 基本不变 |

### 10.3 缓存一致性

**问题**：缓存数据和数据库数据可能不一致

**策略**：
1. **Cache Aside Pattern**（推荐）：先更新数据库，再删除缓存
2. **Write Through**：同时更新缓存和数据库
3. **延迟双删**：更新数据库 → 删除缓存 → 延迟 → 再删除一次

```csharp
// Cache Aside Pattern 实现
public async Task UpdateDataAsync(int id, Data data)
{
    // 1. 先更新数据库
    await _dbContext.SaveChangesAsync();

    // 2. 再删除缓存（而不是更新缓存）
    _cache.Remove($"data:{id}");
    // 下次读取时会自动从数据库加载最新数据到缓存
}
```

### 10.4 内存占用监控

```csharp
// Program.cs 中启用缓存大小跟踪
builder.Services.AddMemoryCache(options =>
{
    options.SizeLimit = 500 * 1024 * 1024;  // 500MB上限
    options.CompactionPercentage = 0.2;      // 当达到80%时开始清理20%
});
```

---

## 11. 完整项目代码结构

```
src/
├── Services/
│   ├── CacheKeys.cs                 # 缓存键常量定义
│   ├── CacheMonitorService.cs       # 缓存监控服务
│   ├── UserProfileService.cs        # 用户信息服务（含缓存）
│   └── BlogPostService.cs           # 文章服务（含缓存）
├── Controllers/
│   ├── UsersController.cs           # 用户控制器
│   └── BlogPostsController.cs       # 文章控制器
├── Models/
│   ├── Dtos/
│   │   ├── UserProfileResponse.cs
│   │   └── BlogPostDto.cs
│   └── PagedResult.cs               # 分页结果类
├── Program.cs                       # 注册 AddMemoryCache()
└── appsettings.json
```

## 12. 常见问题排查 (FAQ)

### Q1: 缓存好像没生效？

**检查清单**：
1. 是否调用了 `AddMemoryCache()`？
2. 注入的接口是否正确（`IMemoryCache` 不是 `IDistributedCache`）？
3. 键名是否一致（注意大小写、空格）？

### Q2: 缓存什么时候会被清除？

**自动清除情况**：
- AbsoluteExpiration 到期
- SlidingExpiration 超过闲置时间
- 内存不足（根据Priority决定淘汰顺序）

**手动清除**：
- 调用 `_cache.Remove(key)`
- 应用程序重启（内存缓存是基于进程的）

### Q3: 内存缓存适合什么场景？

**适合的场景**：
- 单实例部署的应用
- 需要亚毫秒级响应的数据
- 数据量不大（几百MB以内）

**不适合的场景**：
- 多实例/分布式部署（需要Redis等分布式缓存）
- 需要持久化的数据（进程重启会丢失）
- 大数据量缓存（会占用过多内存）

---

## 13. 总结

本教程我们学习了：

✅ **为什么需要缓存** - 加速响应、减轻数据库压力
✅ **IMemoryCache核心API** - GetOrCreateAsync/Set/Get/Remove
✅ **MemoryCacheEntryOptions** - 过期时间、优先级、回调等配置
✅ **缓存键设计规范** - 前缀命名、层次结构
✅ **三大经典问题** - 穿透、击穿、雪崩及解决方案
✅ **三个实战场景** - 用户信息、热门文章、配置缓存
✅ **最佳实践** - 不过期敏感信息、合理设置TTL、监控命中率

**下一步学习**：
- 当应用需要多实例部署时，学习【分布式缓存 Redis】
- 学习【响应缓存与输出缓存】优化HTTP层缓存
- 结合单元测试，测试缓存逻辑的正确性

---

**参考资源**：
- [官方文档：内存缓存](https://docs.microsoft.com/zh-cn/aspnet/core/performance/caching/memory)
- [Microsoft.Extensions.Caching.Memory 源码](https://github.com/dotnet/runtime/tree/main/src/libraries/Microsoft.Extensions.Caching.Memory)

**练习题**：
1. 为你的项目添加用户权限缓存（缓存用户的角色和权限列表）
2. 实现一个带有布隆过滤器的缓存防穿透方案
3. 创建一个缓存监控仪表盘页面，实时显示命中率
