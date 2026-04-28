# 分布式缓存 Redis 完全指南

> **学习目标**：掌握 ASP.NET Core 中 Redis 分布式缓存的完整使用方法，包括环境搭建、基本操作、高级数据类型和生产环境最佳实践
>
> **难度等级**：⭐⭐⭐⭐ 中高级
>
> **前置知识**：IMemoryCache 基础、Docker 基础操作、异步编程
>
> **预计时间**：60分钟

---

## 1. 为什么需要分布式缓存？

### 1.1 内存缓存的局限性

在上一节我们学习了 `IMemoryCache`，它非常适合单实例应用。但在以下场景中会遇到问题：

**[架构图对比]**

**场景1：多实例部署（负载均衡）**
```
                    ┌─ Instance 1 (内存缓存A)
用户请求 → 负载均衡 ─┼─ Instance 2 (内存缓存B)
                    └─ Instance 3 (内存缓存C)

问题：
- 用户第一次请求打到Instance 1，数据缓存到内存A
- 第二次请求打到Instance 2，内存B中没有这个缓存
- 结果：缓存命中率低，每个实例都要查数据库
```

**场景2：应用重启导致缓存丢失**
```
时间线：
T1: 应用启动 → 查数据库 → 缓存到内存
T2: 用户访问 → 缓存命中 ✅ 快速响应
T3: 应用重启/部署新版本 → 内存清空 💥
T4: 用户访问 → 缓存未命中 → 全部打向数据库 ❌
```

### 1.2 分布式缓存的优势

```
                    ┌─ Instance 1 ──┐
用户请求 → 负载均衡 ├─ Instance 2 ──┼──→ Redis Server (共享缓存)
                    └─ Instance 3 ──┘

优势：
✅ 多实例共享同一份缓存
✅ 应用重启不影响缓存数据
✅ 数据持久化（可配置）
✅ 支持更丰富的数据结构
```

---

## 2. Redis 简介

### 2.1 什么是 Redis？

**Redis** (REmote DIctionary Server) 是一个开源的高性能键值对存储数据库。

**核心特点**：

| 特性 | 说明 |
|------|------|
| **速度极快** | 纯内存操作，读写速度 ~10万次/秒 |
| **数据类型丰富** | String、Hash、List、Set、Sorted Set 等 |
| **原子性** | 所有操作都是原子性的 |
| **持久化支持** | RDB快照 + AOF日志 |
| **集群支持** | 主从复制、Sentinel、Cluster |

### 2.2 Redis vs 其他缓存方案

| 方案 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **IMemoryCache** | 单实例、简单场景 | 零配置、最快 | 不共享、不持久化 |
| **Redis** ⭐ | 生产环境推荐 | 功能强大、高性能 | 需要额外部署维护 |
| **Memcached** | 简单的分布式缓存 | 简单易用 | 数据类型单一 |
| **SQL Server Cache** | 已有SQL Server环境 | 无需额外服务 | 性能不如Redis |

---

## 3. 环境搭建与连接配置

### 3.1 使用 Docker 运行 Redis（推荐）

**方式一：快速启动（开发环境）**

```bash
# 拉取并运行Redis镜像
docker run -d --name redis-server -p 6379:6379 redis:latest

# 带密码启动（生产环境必须！）
docker run -d --name redis-server \
  -p 6379:6379 \
  redis:latest \
  redis-server --requirepass "YourStrongPassword123!"
```

**验证 Redis 是否运行成功**：

```bash
# 查看容器状态
docker ps | grep redis

# 应该看到类似输出：
# CONTAINER ID   IMAGE          COMMAND                  STATUS          PORTS
# abc123def456   redis:latest   "docker-entrypoint.s…"   Up 2 hours       0.0.0.0:6379->6379/tcp

# 进入Redis命令行测试
docker exec -it redis-server redis-cli

# 在redis-cli中执行
127.0.0.1:6379> SET test "Hello Redis"
OK
127.0.0.1:6379> GET test
"Hello Redis"
127.0.0.1:6379> EXIT
```

**[截图：Docker Desktop中Redis容器运行状态]**

**方式二：使用 docker-compose.yml（项目集成）**

```yaml
# docker-compose.yml
version: '3.8'
services:
  redis:
    image: redis:latest
    container_name: blog-redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data  # 数据持久化
    command: redis-server --requirepass "YourStrongPassword123!"
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "YourStrongPassword123!", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
```

启动服务：
```bash
docker-compose up -d redis
```

### 3.2 appsettings.json 配置

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BlogDb;Trusted_Connection=True;",
    "Redis": "localhost:6379,password=YourStrongPassword123!"
  },
  "CacheSettings": {
    "AbsoluteExpirationInMinutes": 30,
    "SlidingExpirationInMinutes": 10
  }
}
```

### 3.3 注册分布式缓存服务

在 `Program.cs` 中注册 Redis 缓存：

```csharp
using Microsoft.Extensions.Caching.StackExchangeRedis;

var builder = WebApplication.CreateBuilder(args);

// ✅ 注册 Redis 分布式缓存
var redisConnectionString = builder.Configuration.GetConnectionString("Redis");
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = redisConnectionString;  // 连接字符串
    options.InstanceName = "BlogApp:";              // 键名前缀（可选但推荐！）
});

// 其他服务注册...
builder.Services.AddControllers();
builder.Services.AddDbContext<ApplicationDbContext>(...);

var app = builder.Build();
app.MapControllers();
app.Run();
```

**验证点**：程序启动时如果没有报错，说明 Redis 连接成功。

**⚠️ 如果连接失败会怎样？**
- 应用可以正常启动（Redis不是必需的）
- 但所有缓存操作会抛出异常
- 建议：添加健康检查确认 Redis 可用性

---

## 4. IDistributedCache 接口详解

### 4.1 接口定义

`IDistributedCache` 与 `IMemoryCache` API 非常相似，但有一些关键区别：

```csharp
public interface IDistributedCache
{
    // 获取缓存（返回byte[]而非对象）
    byte[]? Get(string key);

    // 异步获取
    Task<byte[]?> GetAsync(string key, CancellationToken token = default);

    // 设置缓存（需要手动序列化为byte[]）
    void Set(string key, byte[] value, DistributedCacheEntryOptions options);
    Task SetAsync(string key, byte[] value, DistributedCacheEntryOptions options,
        CancellationToken token = default);

    // 刷新缓存（重置滑动过期时间）
    void Refresh(string key);
    Task RefreshAsync(string key, CancellationToken token = default);

    // 移除缓存
    void Remove(string key);
    Task RemoveAsync(string key, CancellationToken token = default);
}
```

**关键区别**：

| 特性 | IMemoryCache | IDistributedCache |
|------|-------------|-------------------|
| **数据类型** | 直接存储对象 `T` | 存储 `byte[]`（需要序列化） |
| **API风格** | 同步 + 异步 | 主要是异步 |
| **GetOrCreate** | ✅ 有 | ❌ 没有（需自己实现） |
| **作用域** | 进程内 | 跨进程/跨服务器 |
| **过期选项** | MemoryCacheEntryOptions | DistributedCacheEntryOptions |

### 4.2 序列化辅助工具类

由于 `IDistributedCache` 只能存储 `byte[]`，我们需要一个序列化工具类：

```csharp
using System.Text.Json;

/// <summary>
/// Redis缓存序列化辅助类
/// </summary>
public static class CacheSerializer
{
    private static readonly JsonSerializerOptions _jsonOptions = new()
    {
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
        WriteIndented = false
    };

    /// <summary>
    /// 对象序列化为byte[]
    /// </summary>
    public static byte[] ToBytes<T>(T value)
    {
        return JsonSerializer.SerializeToUtf8Bytes(value, _jsonOptions);
    }

    /// <summary>
    /// byte[]反序列化为对象
    /// </summary>
    public static T? FromBytes<T>(byte[] bytes)
    {
        return JsonSerializer.Deserialize<T>(bytes, _jsonOptions);
    }

    /// <summary>
    /// byte[]反序列化为对象（异步版本）
    /// </summary>
    public static async Task<T?> FromBytesAsync<T>(byte[] bytes)
    {
        await using var stream = new MemoryStream(bytes);
        return await JsonSerializer.DeserializeAsync<T>(stream, _jsonOptions);
    }
}
```

---

## 5. 基本使用示例

### 5.1 封装 Redis 缓存服务

为了使用方便，我们封装一个通用的 Redis 缓存服务：

```csharp
using Microsoft.Extensions.Caching.Distributed;
using System.Text.Json;

public interface IRedisCacheService
{
    /// <summary>
    /// 获取缓存（泛型版本）
    /// </summary>
    Task<T?> GetAsync<T>(string key);

    /// <summary>
    /// 设置缓存
    /// </summary>
    Task SetAsync<T>(string key, T value, TimeSpan? absoluteExpiration = null);

    /// <summary>
    /// 获取或创建缓存（类似IMemoryCache的GetOrCreate）
    /// </summary>
    Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory,
        TimeSpan? absoluteExpiration = null);

    /// <summary>
    /// 移除缓存
    /// </summary>
    Task RemoveAsync(string key);

    /// <summary>
    /// 刷新缓存（重置滑动过期时间）
    /// </summary>
    Task RefreshAsync(string key);

    /// <summary>
    /// 检查键是否存在
    /// </summary>
    Task<bool> ExistsAsync(string key);
}

public class RedisCacheService : IRedisCacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<RedisCacheService> _logger;
    private readonly DistributedCacheEntryOptions _defaultOptions;

    public RedisCacheService(
        IDistributedCache cache,
        ILogger<RedisCacheService> logger,
        IConfiguration configuration)
    {
        _cache = cache;
        _logger = logger;

        // 从配置读取默认过期时间
        var absoluteExpirationMinutes = configuration.GetValue<int>(
            "CacheSettings:AbsoluteExpirationInMinutes", 30);

        _defaultOptions = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(absoluteExpirationMinutes)
        };
    }

    public async Task<T?> GetAsync<T>(string key)
    {
        try
        {
            var bytes = await _cache.GetAsync(key);
            if (bytes == null)
            {
                _logger.LogDebug("缓存未命中: {Key}", key);
                return default;
            }

            _logger.LogDebug("缓存命中: {Key}", key);
            return CacheSerializer.FromBytes<T>(bytes);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "读取缓存失败: {Key}", key);
            return default;  // 缓存失败不应影响业务
        }
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan? absoluteExpiration = null)
    {
        try
        {
            var options = absoluteExpiration.HasValue
                ? new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = absoluteExpiration.Value }
                : _defaultOptions;

            var bytes = CacheSerializer.ToBytes(value);
            await _cache.SetAsync(key, bytes, options);

            _logger.LogDebug("缓存已设置: {Key}, 过期时间: {Expiration}",
                key, options.AbsoluteExpirationRelativeToNow);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "设置缓存失败: {Key}", key);
        }
    }

    public async Task<T> GetOrCreateAsync<T>(string key, Func<Task<T>> factory,
        TimeSpan? absoluteExpiration = null)
    {
        // 尝试从缓存获取
        var cached = await GetAsync<T>(key);
        if (cached != null && !cached.Equals(default(T)))
        {
            return cached;
        }

        // 缓存不存在，执行工厂方法
        _logger.LogInformation("缓存未命中，执行数据加载: {Key}", key);
        var result = await factory();

        // 存入缓存
        if (result != null)
        {
            await SetAsync(key, result, absoluteExpiration);
        }

        return result;
    }

    public async Task RemoveAsync(string key)
    {
        await _cache.RemoveAsync(key);
        _logger.LogDebug("缓存已移除: {Key}", key);
    }

    public async Task RefreshAsync(string key)
    {
        await _cache.RefreshAsync(key);
    }

    public async Task<bool> ExistsAsync(string key)
    {
        var value = await _cache.GetAsync(key);
        return value != null;
    }
}
```

**注册服务**：

```csharp
// Program.cs
builder.Services.AddSingleton<IRedisCacheService, RedisCacheService>();
```

### 5.2 在业务代码中使用

**示例：用户信息缓存**

```csharp
public class UserService
{
    private readonly IRedisCacheService _cache;
    private readonly ApplicationDbContext _dbContext;

    public UserService(IRedisCacheService cache, ApplicationDbContext dbContext)
    {
        _cache = cache;
        _dbContext = dbContext;
    }

    public async Task<UserProfileDto?> GetUserProfileAsync(int userId)
    {
        string cacheKey = $"user:{userId}:profile";

        return await _cache.GetOrCreateAsync(cacheKey, async () =>
        {
            Console.WriteLine("[Redis缓存未命中] 从数据库查询...");

            var user = await _dbContext.Users
                .Include(u => u.Role)
                .FirstOrDefaultAsync(u => u.Id == userId);

            if (user == null) return null;

            return new UserProfileDto
            {
                Id = user.Id,
                Email = user.Email,
                Nickname = user.Nickname,
                RoleName = user.Role?.Name
            };
        }, TimeSpan.FromMinutes(30));  // 缓存30分钟
    }

    public async Task UpdateProfileAsync(int userId, UpdateProfileDto dto)
    {
        var user = await _dbContext.Users.FindAsync(userId);
        user.Nickname = dto.Nickname;
        user.Bio = dto.Bio;
        await _dbContext.SaveChangesAsync();

        // ✅ 更新后清除缓存
        await _cache.RemoveAsync($"user:{userId}:profile");
    }
}
```

**Controller 使用**：

```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly UserService _userService;

    public UsersController(UserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{userId}/profile")]
    public async Task<ActionResult<UserProfileDto>> GetUserProfile(int userId)
    {
        var profile = await _userService.GetUserProfileAsync(userId);
        if (profile == null)
            return NotFound(new { message = "用户不存在" });

        return Ok(profile);
    }
}
```

**验证步骤**：
1. 启动应用和 Redis
2. 调用 GET /api/users/1/profile - 观察控制台输出"从数据库查询..."
3. 再次调用同一接口 - 控制台无输出（Redis缓存命中）
4. 打开 Redis CLI 执行 `KEYS *` 可以看到缓存的数据
5. 在另一个终端或Postman再次调用 - 同样命中缓存（证明是分布式的）

---

## 6. Redis 高级数据类型直接操作

虽然 `IDistributedCache` 只提供了简单的 Key-Value 操作，但有时候我们需要使用 Redis 的更高级数据结构。这时可以使用 `StackExchange.Redis` 库。

### 6.1 安装 StackExchange.Redis

```bash
dotnet add package StackExchange.Redis
```

### 6.2 创建 Redis 连接管理器

```csharp
using StackExchange.Redis;

public interface IRedisConnectionMultiplexer
{
    IDatabase GetDatabase(int db = -1);
    IServer GetServer(EndPoint endPoint);
    ISubscriber GetSubscriber();
}

public class RedisConnectionMultiplexer : IRedisConnectionMultiplexer
{
    private readonly ConnectionMultiplexer _connection;

    public RedisConnectionMultiplexer(IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("Redis");
        _connection = ConnectionMultiplexer.Connect(connectionString);
    }

    public IDatabase GetDatabase(int db = -1)
    {
        return _connection.GetDatabase(db);
    }

    public IServer GetServer(EndPoint endPoint)
    {
        return _connection.GetServer(endPoint);
    }

    public ISubscriber GetSubscriber()
    {
        return _connection.GetSubscriber();
    }
}

// 注册为单例
builder.Services.AddSingleton<IRedisConnectionMultiplexer, RedisConnectionMultiplexer>();
```

### 6.3 五大数据类型实战

#### 6.3.1 String 类型（字符串）- 最常用

**适用场景**：简单缓存、计数器、Session、Token

```csharp
public class StringOperationsExample
{
    private readonly IDatabase _db;

    public StringOperationsExample(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 示例1：基本读写
    /// </summary>
    public async Task BasicStringExample()
    {
        // SET
        await _db.StringSetAsync("user:1:name", "张三");

        // GET
        var name = await _db.StringGetAsync("user:1:name");
        Console.WriteLine(name);  // 输出: 张三

        // SET with TTL（带过期时间）
        await _db.StringSetAsync("temp:code", "123456",
            expiry: TimeSpan.FromMinutes(5));  // 5分钟后自动删除
    }

    /// <summary>
    /// 示例2：计数器（原子操作）
    /// </summary>
    public async Task CounterExample()
    {
        // 文章浏览次数 +1（原子递增）
        long viewCount = await _db.StringIncrementAsync($"post:1:views");
        Console.WriteLine($"当前浏览数: {viewCount}");

        // 可以指定增量
        await _db.StringIncrementAsync("post:1:likes", 10);  // +10

        // 递减
        await _db.StringDecrementAsync("inventory:product:1", 1);  // 库存-1
    }

    /// <summary>
    /// 示例3：限流计数器（滑动窗口算法）
    /// </summary>
    public async Task<bool> RateLimitCheckAsync(string userId, int maxRequests = 100)
    {
        string key = $"ratelimit:{userId}:{DateTime.UtcNow:yyyyMMddHHmm}";
        var currentCount = await _db.StringIncrementAsync(key);

        if (currentCount == 1)
        {
            // 第一次访问，设置1分钟过期
            await _db.KeyExpireAsync(key, TimeSpan.FromMinutes(1));
        }

        return currentCount <= maxRequests;  // true = 允许访问
    }
}
```

#### 6.3.2 Hash 类型（哈希）- 对象属性

**适用场景**：对象存储、用户Profile、商品信息

```csharp
public class HashOperationsExample
{
    private readonly IDatabase _db;

    public HashOperationsExample(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 存储用户对象（使用Hash）
    /// </summary>
    public async Task StoreUserHashAsync(int userId, UserDto user)
    {
        string key = $"user:{userId}";

        var entries = new HashEntry[]
        {
            new HashEntry("id", userId),
            new HashEntry("email", user.Email),
            new HashEntry("nickname", user.Nickname),
            new HashEntry("avatar", user.Avatar ?? ""),
            new HashEntry("role", user.Role),
            new HashEntry("created_at", user.CreatedAt.ToString("o"))
        };

        await _db.HashSetAsync(key, entries);
        await _db.KeyExpireAsync(key, TimeSpan.FromHours(1));
    }

    /// <summary>
    /// 获取用户对象
    /// </summary>
    public async Task<UserDto?> GetUserHashAsync(int userId)
    {
        string key = $"user:{userId}";
        var hash = await _db.HashGetAllAsync(key);

        if (hash.Length == 0) return null;

        return new UserDto
        {
            Id = int.Parse(hash.FirstOrDefault(h => h.Name == "id").Value),
            Email = hash.FirstOrDefault(h => h.Name == "email").Value,
            Nickname = hash.FirstOrDefault(h => h.Name == "nickname").Value,
            Avatar = hash.FirstOrDefault(h => h.Name == "avatar").Value,
            Role = hash.FirstOrDefault(h => h.Name == "role").Value
        };
    }

    /// <summary>
    /// 只更新单个字段（比更新整个对象高效）
    /// </summary>
    public async Task UpdateUserNicknameAsync(int userId, string nickname)
    {
        await _db.HashSetAsync($"user:{userId}", "nickname", nickname);
    }

    /// <summary>
    /// 检查字段是否存在
    /// </summary>
    public async Task<bool> UserFieldExists(int userId, string field)
    {
        return await _db.HashExistsAsync($"user:{userId}", field);
    }
}
```

#### 6.3.3 List 类型（列表）- 队列/栈

**适用场景**：消息队列、最新列表、任务队列

```csharp
public class ListOperationsExample
{
    private readonly IDatabase _db;

    public ListOperationsExample(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 最新评论队列（保留最新20条）
    /// </summary>
    public async Task AddRecentCommentAsync(int postId, CommentDto comment)
    {
        string key = $"post:{postId}:recent_comments";

        // LPUSH：从左侧压入（最新的在最前面）
        await _db.ListLeftPushAsync(key,
            JsonSerializer.Serialize(comment));

        // LTRIM：只保留最新的20条
        await _db.ListTrimAsync(key, 0, 19);

        // 设置过期时间
        await _db.KeyExpireAsync(key, TimeSpan.FromHours(24));
    }

    /// <summary>
    /// 获取最新评论
    /// </summary>
    public async Task<List<CommentDto>> GetRecentCommentsAsync(int postId)
    {
        string key = $"post:{postId}:recent_comments";
        var values = await _db.RangeAsync(key, 0, -1);  // 获取全部

        return values.Select(v =>
            JsonSerializer.Deserialize<CommentDto>(v)).ToList()!;
    }

    /// <summary>
    /// 任务队列（FIFO）
    /// </summary>
    public async Task EnqueueTaskAsync(TaskItem task)
    {
        // RPUSH：从右侧入队（队尾）
        await _db.ListRightPushAsync("task_queue",
            JsonSerializer.Serialize(task));
    }

    public async Task<TaskItem?> DequeueTaskAsync()
    {
        // LPOP：从左侧出队（队头）
        var value = await _db.ListLeftPopAsync("task_queue");
        return value.IsNullOrEmpty
            ? null
            : JsonSerializer.Deserialize<TaskItem>(value);
    }
}
```

#### 6.3.4 Set 类型（集合）- 去重

**适用场景**：标签、好友列表、去重、交集/并集运算

```csharp
public class SetOperationsExample
{
    private readonly IDatabase _db;

    public SetOperationsExample(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 文章标签集合
    /// </summary>
    public async Task AddTagsToPostAsync(int postId, params string[] tags)
    {
        string key = $"post:{postId}:tags";
        var redisValues = tags.Select(t => (RedisValue)t).ToArray();

        // SADD：添加到集合（自动去重）
        await _db.SetAddAsync(key, redisValues);
    }

    /// <summary>
    /// 获取文章的所有标签
    /// </summary>
    public async Task<string[]> GetPostTagsAsync(int postId)
    {
        var members = await _db.SetMembersAsync($"post:{postId}:tags");
        return members.Select(m => m.ToString()).ToArray();
    }

    /// <summary>
    /// 用户点赞（防止重复点赞）
    /// </summary>
    public async Task<bool> LikePostAsync(int userId, int postId)
    {
        string key = $"post:{postId}:liked_users";

        // SISMEMBER：检查是否已存在
        bool alreadyLiked = await _db.SetContainsAsync(key, userId);

        if (alreadyLiked)
        {
            return false;  // 已经点赞过了
        }

        // SADD：添加点赞记录
        await _db.SetAddAsync(key, userId);

        // 点赞数+1
        await _db.StringIncrementAsync($"post:{postId}:likes");

        return true;  // 点赞成功
    }

    /// <summary>
    /// 集合运算示例：找出两个标签的共同文章
    /// </summary>
    public async Task<RedisValue[]> FindCommonPostsByTags(string tag1, string tag2)
    {
        // SINTER：求交集
        return await _db.SetCombineAsync(
            SetOperation.Intersect,
            $"tag:{tag1}:posts",
            $"tag:{tag2}:posts");
    }
}
```

#### 6.3.5 Sorted Set 类型（有序集合）- 排行榜

**适用场景**：排行榜、评分系统、时间线排序

```csharp
public class SortedSetOperationsExample
{
    private readonly IDatabase _db;

    public SortedSetOperationsExample(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 文章热度排行榜（按浏览量排序）
    /// </summary>
    public async Task UpdatePostRankingAsync(int postId, long viewCount)
    {
        // ZADD：添加到有序集合（score = viewCount）
        await _db.SortedSetAddAsync(
            "ranking:posts:views",
            postId,     // member
            viewCount); // score
    }

    /// <summary>
    /// 获取Top N热门文章
    /// </summary>
    public async Task<List<PostRankingDto>> GetTopPostsAsync(int topN = 10)
    {
        // ZREVRANGE：按分数降序获取（从高到低）
        var entries = await _db.SortedSetRangeByRankWithScoresAsync(
            "ranking:posts:views",
            order: Order.Descending,  // 降序
            start: 0,
            stop: topN - 1);         // 取Top N

        return entries.Select(e => new PostRankingDto
        {
            PostId = (int)e.Element,
            ViewCount = (long)e.Score
        }).ToList();
    }

    /// <summary>
    /// 获取某篇文章的排名
    /// </summary>
    public async Task<long?> GetPostRankAsync(int postId)
    {
        // ZREVRANK：获取排名（从0开始，分数越高排名越靠前数值越小）
        var rank = await _db.SortedSetRankAsync(
            "ranking:posts:views",
            postId,
            Order.Descending);

        return rank?.HasValue == true ? rank.Value + 1 : null;  // +1转为第几名
    }

    /// <summary>
    /// 用户积分排行榜
    /// </summary>
    public async Task AddUserScoreAsync(int userId, double scoreDelta)
    {
        // ZINCRBY：增加分数
        await _db.SortedSetIncrementAsync(
            "ranking:user:scores",
            userId,
            scoreDelta);
    }

    /// <summary>
    /// 获取用户排名和分数范围
    /// </summary>
    public async Task<UserLeaderboardDto> GetUserLeaderboardInfo(int userId)
    {
        var rank = await _db.SortedSetRankAsync(
            "ranking:user:scores",
            userId,
            Order.Descending);

        var score = await _db.SortedSetScoreAsync(
            "ranking:user:scores",
            userId);

        return new UserLeaderboardDto
        {
            UserId = userId,
            Rank = rank?.HasValue == true ? (int)(rank.Value + 1) : -1,
            Score = score ?? 0
        };
    }
}
```

---

## 7. 完整实战案例

### 7.1 Session 共享（多实例登录状态同步）

```csharp
/// <summary>
/// 分布式Session服务
/// </summary>
public class DistributedSessionService
{
    private readonly IDatabase _db;
    private const int SessionExpiryHours = 24;

    public DistributedSessionService(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 创建Session（登录成功后调用）
    /// </summary>
    public async Task<string> CreateSessionAsync(int userId, UserInfo userInfo)
    {
        string sessionId = GenerateSessionId();
        string key = $"session:{sessionId}";

        var sessionData = new
        {
            UserId = userId,
            Email = userInfo.Email,
            Nickname = userInfo.Nickname,
            Role = userInfo.Role,
            LoginTime = DateTime.UtcNow,
            IpAddress = userInfo.IpAddress
        };

        // 存储Session数据（Hash结构）
        var entries = new HashEntry[]
        {
            new ("user_id", userId),
            new ("email", userInfo.Email),
            new ("nickname", userInfo.Nickname),
            new ("role", userInfo.Role),
            new ("login_time", DateTime.UtcNow.ToString("o")),
            new ("ip", userInfo.IpAddress)
        };

        await _db.HashSetAsync(key, entries);
        await _db.KeyExpireAsync(key, TimeSpan.FromHours(SessionExpiryHours));

        return sessionId;
    }

    /// <summary>
    /// 获取Session（每次请求时调用）
    /// </summary>
    public async Task<SessionData?> GetSessionAsync(string sessionId)
    {
        string key = $"session:{sessionId}";
        var hash = await _db.HashGetAllAsync(key);

        if (hash.Length == 0) return null;

        // 续期（滑动过期）
        await _db.KeyExpireAsync(key, TimeSpan.FromHours(SessionExpiryHours));

        return new SessionData
        {
            UserId = int.Parse(hash.First(h => h.Name == "user_id").Value),
            Email = hash.First(h => h.Name == "email").Value,
            Nickname = hash.First(h => h.Name == "nickname").Value,
            Role = hash.First(h => h.Name == "role").Value
        };
    }

    /// <summary>
    /// 注销Session（登出时调用）
    /// </summary>
    public async Task RemoveSessionAsync(string sessionId)
    {
        await _db.KeyDeleteAsync($"session:{sessionId}");
    }

    private string GenerateSessionId()
    {
        return Guid.NewGuid().ToString("N");
    }
}
```

### 7.2 API 限流计数器

```csharp
/// <summary>
/// 基于Redis的限流中间件
/// </summary>
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IDatabase _redisDb;

    public RateLimitingMiddleware(RequestDelegate next, IRedisConnectionMultiplexer redis)
    {
        _next = next;
        _redisDb = redis.GetDatabase();
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        var endpoint = context.Request.Path.ToString();

        // 滑动窗口限流：每分钟最多100次请求
        string key = $"ratelimit:{clientId}:{endpoint}:{DateTime.UtcNow:yyyyMMddHHmm}";

        var currentCount = await _redisDb.StringIncrementAsync(key);

        if (currentCount == 1)
        {
            await _redisDb.KeyExpireAsync(key, TimeSpan.FromMinutes(1));
        }

        // 设置响应头（让客户端知道限额）
        context.Response.Headers["X-RateLimit-Limit"] = "100";
        context.Response.Headers["X-RateLimit-Remaining"] =
            Math.Max(0, 100 - currentCount).ToString();

        if (currentCount > 100)
        {
            context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
            await context.Response.WriteAsJsonAsync(new
            {
                error = "Too many requests",
                message = "请稍后再试",
                retryAfterSeconds = 60
            });
            return;
        }

        await _next(context);
    }
}

// 注册中间件
// app.UseMiddleware<RateLimitingMiddleware>();
```

### 7.3 实时排行榜系统

```csharp
[ApiController]
[Route("api/[controller]")]
public class LeaderboardController : ControllerBase
{
    private readonly IDatabase _db;

    public LeaderboardController(IRedisConnectionMultiplexer redis)
    {
        _db = redis.GetDatabase();
    }

    /// <summary>
    /// 获取文章热度排行榜
    /// GET /api/leaderboard/posts?type=views&top=10
    /// </summary>
    [HttpGet("posts")]
    public async Task<ActionResult<object>> GetPostLeaderboard(
        [FromQuery] string type = "views",
        [FromQuery] int top = 10)
    {
        string key = type switch
        {
            "views" => "ranking:posts:views",
            "likes" => "ranking:posts:likes",
            "comments" => "ranking:posts:comments",
            _ => "ranking:posts:views"
        };

        var entries = await _db.SortedSetRangeByRankWithScoresAsync(
            key,
            order: Order.Descending,
            start: 0,
            stop: top - 1);

        var result = entries.Select((e, index) => new
        {
            Rank = index + 1,
            PostId = (int)e.Element,
            Score = (long)e.Score
        });

        return Ok(new
        {
            Type = type,
            Top = top,
            Items = result
        });
    }

    /// <summary>
    /// 获取我的排名
    /// GET /api/leaderboard/my-rank?postId=123&type=views
    /// </summary>
    [HttpGet("my-rank")]
    public async Task<ActionResult<object>> GetMyRank(
        [FromQuery] int postId,
        [FromQuery] string type = "views")
    {
        string key = $"ranking:posts:{type}";

        var rank = await _db.SortedSetRankAsync(key, postId, Order.Descending);
        var score = await _db.SortedSetScoreAsync(key, postId);

        if (!rank.HasValue)
        {
            return NotFound(new { message = "该文章暂无排行数据" });
        }

        return Ok(new
        {
            PostId = postId,
            Rank = rank.Value + 1,  // 转换为第几名
            Score = score
        });
    }
}
```

---

## 8. Redis 持久化和集群简介

### 8.1 持久化方案

**RDB (Redis Database)** - 快照持久化
```bash
# redis.conf 配置
save 900 1      # 900秒内至少有1个key变化就保存
save 300 10     # 300秒内至少有10个key变化就保存
save 60 10000   # 60秒内至少有10000个key变化就保存

dbfilename dump.rdb
dir /data
```

**AOF (Append Only File)** - 日志持久化
```bash
# redis.conf 配置
appendonly yes
appendfsync everysec  # 每秒同步一次（推荐）
# appendfsync always  # 每次写操作都同步（最安全但最慢）
# appendfsync no      # 由操作系统决定（最快但可能丢失数据）
```

**生产环境建议**：同时开启 RDB + AOF

### 8.2 高可用方案

| 方案 | 复杂度 | 说明 |
|------|--------|------|
| **主从复制** | 低 | 一主多从，读写分离 |
| **Sentinel**（哨兵） | 中 | 自动故障转移 |
| **Cluster**（集群） | 高 | 数据分片，水平扩展 |

**Sentinel 配置示例（docker-compose）**：
```yaml
services:
  redis-master:
    image: redis:latest
    command: redis-server --requirepass "password"

  redis-slave:
    image: redis:latest
    command: redis-server --slaveof redis-master 6379 --requirepass "password" --masterauth "password"

  redis-sentinel:
    image: redis:latest
    command: redis-sentinel /etc/redis/sentinel.conf
```

---

## 9. 生产环境安全配置

### 9.1 必要的安全措施

```bash
# 1. 设置强密码
redis-server --requirepass "YourVeryStrong&ComplexPassword!2024"

# 2. 重命名危险命令（禁用或重命名）
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG ""
rename-command DEBUG ""

# 3. 绑定特定IP（不要绑定0.0.0.0）
bind 127.0.0.1  # 仅本地访问
# 或 bind 192.168.1.100  # 绑定内网IP

# 4. 设置最大内存
maxmemory 2gb
maxmemory-policy allkeys-lru  # 内存满时的淘汰策略
```

### 9.2 连接字符串最佳实践

```json
// appsettings.Production.json
{
  "ConnectionStrings": {
    "Redis": "redis-production:6379,password=YourStrongPassword!,ssl=true,abortConnect=false"
  }
}
```

**连接参数说明**：

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `ssl=true` | 启用TLS加密 | 生产环境必须 |
| `abortConnect=false` | 连接失败时不抛异常 | 提高可用性 |
| `connectRetry=3` | 重试次数 | 3次 |
| `connectTimeout=5000` | 连接超时(ms) | 5000 |
| `syncTimeout=5000` | 操作超时(ms) | 5000 |

---

## 10. 监控 Redis

### 10.1 Redis CLI 常用监控命令

```bash
# 连接Redis
docker exec -it redis-server redis-cli -a password

# 基本信息
INFO server        # 服务器信息
INFO memory         # 内存使用情况
INFO stats          # 统计信息
INFO replication    # 主从复制状态
INFO clients        # 客户端连接信息

# 实时监控
MONITOR             # 监控所有执行的命令（调试用）

# 慢查询日志
SLOWLOG GET 10      # 获取最近10条慢查询

# 键空间统计
DBSIZE              # 当前数据库的key数量
KEYS *              # 列出所有key（慎用，数据量大时会阻塞）
SCAN 0 COUNT 100    # 游标遍历key（推荐）

# 查看某个key的类型
TYPE key_name
```

### 10.2 RedisInsight 图形化工具（推荐）

**RedisInsight** 是 Redis 官方提供的免费图形化管理工具。

**安装方式**：
```bash
# Docker方式运行
docker run -d --name redisinsight -p 8001:8001 redis/redisinsight:latest
```

然后浏览器访问 `http://localhost:8001`

**功能特性**：
- 浏览器和编辑数据
- 查看性能指标图表
- 慢查询分析
- 命令行界面
- 连接多个Redis实例

**[截图：RedisInsight Dashboard界面]**

### 10.3 应用层监控代码

```csharp
/// <summary>
/// Redis健康检查端点
/// </summary>
public class RedisHealthCheck : IHealthCheck
{
    private readonly IDistributedCache _cache;

    public RedisHealthCheck(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // 尝试写入和读取
            var testKey = $"healthcheck:{Guid.NewGuid()}";
            await _cache.SetAsync(testKey,
                Encoding.UTF8.GetBytes("ok"),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(5)
                });

            var value = await _cache.GetAsync(testKey);
            await _cache.RemoveAsync(testKey);

            if (value != null)
            {
                return HealthCheckResult.Healthy("Redis连接正常");
            }

            return HealthCheckResult.Degraded("Redis响应异常");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy($"Redis连接失败: {ex.Message}");
        }
    }
}

// Program.cs 注册健康检查
builder.Services.AddHealthChecks()
    .AddCheck<RedisHealthCheck>("redis");

// 健康检查端点
app.MapHealthChecks("/health");
```

**访问 `/health` 端点的响应示例**：
```json
{
  "status": "Healthy",
  "totalDuration": "00:00:00.0234506",
  "entries": {
    "redis": {
      "status": "Healthy",
      "description": "Redis连接正常",
      "duration": "00:00:00.0212345"
    }
  }
}
```

---

## 11. 常见问题与解决方案

### Q1: 连接超时怎么办？

**排查步骤**：
1. 检查 Redis 服务是否正常运行：`docker ps | grep redis`
2. 检查防火墙是否开放 6379 端口
3. 检查连接字符串是否正确（地址、密码）
4. 增加 `connectTimeout` 参数

```csharp
options.Configuration = "localhost:6379,password=xxx,connectTimeout=10000,syncTimeout=10000";
```

### Q2: 内存不足怎么办？

**解决方案**：
```bash
# 1. 设置最大内存
maxmemory 2gb

# 2. 设置淘汰策略
maxmemory-policy allkeys-lru  # 最近最少使用的先淘汰
# 或 volatile-lru           # 只淘汰有过期时间的key
# 或 allkeys-random         # 随机淘汰
```

### Q3: 如何处理 Redis 不可用的情况？

**优雅降级策略**：

```csharp
public async Task<T?> GetWithFallbackAsync<T>(string key, Func<Task<T>> fallback)
{
    try
    {
        return await _redisCache.GetAsync<T>(key);
    }
    catch (Exception ex)
    {
        _logger.LogWarning(ex, "Redis不可用，回退到数据库查询");
        return await fallback();  // 回退到数据库
    }
}
```

---

## 12. 项目文件清单

```
src/
├── Services/
│   ├── CacheSerializer.cs                 # 序列化辅助类
│   ├── IRedisCacheService.cs              # 缓存服务接口
│   ├── RedisCacheService.cs               # 缓存服务实现
│   ├── IRedisConnectionMultiplexer.cs     # Redis连接接口
│   ├── RedisConnectionMultiplexer.cs      # Redis连接实现
│   ├── DistributedSessionService.cs       # 分布式Session
│   └── RateLimitingMiddleware.cs          # 限流中间件
├── Controllers/
│   ├── LeaderboardController.cs           # 排行榜控制器
│   └── HealthController.cs                # 健康检查控制器
├── Models/
│   ├── Dtos/
│   │   ├── UserProfileDto.cs
│   │   ├── UserDto.cs
│   │   ├── PostRankingDto.cs
│   │   └── SessionData.cs
│   └── Entities/
│       └── User.cs
├── Program.cs                             # 注册服务
├── appsettings.json                       # 开发环境配置
├── appsettings.Production.json            # 生产环境配置
└── docker-compose.yml                     # Redis容器编排
```

---

## 13. 总结

本教程我们全面学习了：

✅ **为什么需要分布式缓存** - 多实例共享、数据持久化
✅ **Redis环境搭建** - Docker部署、连接配置
✅ **IDistributedCache接口** - 基本缓存操作
✅ **五大数据类型** - String/Hash/List/Set/Sorted Set及实战
✅ **完整案例** - Session共享、限流计数器、排行榜系统
✅ **生产实践** - 安全配置、监控运维、高可用方案

**性能对比参考**：

| 操作 | IMemoryCache | Redis (本地网络) | Redis (同机房) |
|------|-------------|------------------|----------------|
| 读 | ~0.001ms | ~0.5ms | ~1ms |
| 写 | ~0.001ms | ~0.5ms | ~1ms |
| 适用场景 | 单实例 | 开发/测试 | 生产环境 |

**下一步学习**：
- 学习【响应缓存与输出缓存】优化HTTP层
- 学习【单元测试】测试缓存逻辑的正确性
- 深入学习 Redis Cluster 集群部署

---

**参考资源**：
- [Redis官方文档](https://redis.io/documentation)
- [StackExchange.Redis文档](https://stackexchange.github.io/StackExchange.Redis/)
- [Microsoft.Extensions.Caching.StackExchangeRedis](https://docs.microsoft.com/zh-cn/aspnet/core/performance/caching/distributed)

**练习题**：
1. 为博客系统实现基于 Redis 的文章浏览量实时统计
2. 实现一个基于 Sorted Set 的"每日签到"功能
3. 使用 Redis 的发布订阅功能实现简单的消息通知系统
