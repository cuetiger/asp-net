# CloudMall 微服务电商商城

> **对应教程**：[[高手篇/07-电商商城实战]]
> **难度等级**：⭐⭐⭐⭐⭐ 专家级 | **预计耗时**：30-40小时
> **适用人群**：已完成博客系统项目，希望掌握微服务架构、分布式事务、消息队列和容器编排的高级开发者

---

## 项目概述

CloudMall 是一个 **B2C 微服务电商平台**，采用领域驱动设计（DDD）思想，将传统单体电商应用拆分为 **8 个独立部署的微服务**，每个服务拥有独立的数据库、独立的代码库和独立的部署周期。通过 RabbitMQ 消息队列实现服务间异步通信，使用 Saga 模式管理分布式事务，借助 YARP 反向代理统一 API 入口，最终通过 Docker + Docker Compose 实现一键容器化部署。

这不仅仅是一个电商 Demo，而是一个**生产级微服务架构的完整参考实现**。它涵盖了微服务架构设计中的核心挑战：服务拆分边界、分布式数据一致性、服务发现与负载均衡、熔断降级、链路追踪、日志聚合、监控告警等。

**核心价值**：
- 掌握微服务架构的设计原则与实施方法
- 理解分布式事务的 Saga 模式与补偿机制
- 学会消息队列（RabbitMQ）在微服务中的实际应用
- 掌握 Docker 容器化与 Docker Compose 编排
- 建立企业级 DevOps 和可观测性意识

---

## 技术栈

### 后端技术栈

| 技术 | 版本 | 用途说明 | 应用服务 |
|------|------|----------|---------|
| **ASP.NET Core Web API** | 8.0 | 各微服务的 HTTP API 框架 | 全部8个服务 |
| **Worker Service** | 8.0 | 后台任务处理（订单超时取消、库存释放等） | Order/Inventory/Notification |
| **Entity Framework Core** | 8.0 | ORM 框架（每服务独立数据库） | 全部服务 |
| **PostgreSQL** | 16.x | 主数据库（4个独立实例） | Product/Order/Cart/Identity |
| **Redis** | 7.0+ | 缓存 + 购物车存储 + 分布式锁 + 会话 | Product/Cart/Identity |
| **RabbitMQ** | 3.12.x | 消息队列（服务间异步通信 + 事件总线） | 全部服务 |
| **Elasticsearch** | 8.x | 商品全文搜索 | Product Service |
| **MinIO** | 最新版 | S3 兼容对象存储（商品图片） | Product Service |
| **YARP (Yet Another Reverse Proxy)** | 2.x | API 网关（路由转发/聚合/限流/认证） | API Gateway |
| **JWT (RSA)** | System.IdentityModel.Tokens.Jwt 7.x | RSA 非对称签名 Token 认证 | Identity/Gateway |
| **OAuth2 / OpenID Connect** | IdentityServer4 / Duende | 第三方登录（GitHub/Google/微信） | Identity Service |
| **Polly** | 8.x | 弹性策略（重试/熔断/舱壁隔离/超时） | 全部服务（HttpClient） |
| **MassTransit** | 8.x | RabbitMQ 抽象层（简化消息发布/消费） | 全部服务 |
| **Serilog** | 3.x | 结构化日志（JSON 格式，便于 ELK 聚合） | 全部服务 |
| **OpenTelemetry .NET** | 1.x | 分布式链路追踪（Jaeger/Zipkin 集成） | 全部服务 |
| **Prometheus.Client** | 最新版 | metrics 指标暴露（/metrics 端点） | 全部服务 |
| **FluentValidation** | 11.x | 请求验证 | 全部服务 |
| **AutoMapper** | 12.x | 对象映射 | 全部服务 |
| **Swashbuckle** | 6.x | Swagger 文档（各服务独立文档） | 全部服务 |
| **xUnit + Moq** | 最新版 | 单元测试 | 全部服务 |

### 前端技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **Vue.js 3** | 3.4+ | 前端框架（Composition API） |
| **Vite** | 5.x | 构建工具 |
| **Pinia** | 2.x | 状态管理 |
| **Vue Router 4** | 4.x | 路由 |
| **Axios** | 1.x | HTTP 客户端（通过网关访问后端） |
| **Element Plus** | 2.5.x | UI 组件库 |
| **ECharts** | 5.x | 数据可视化（销售报表/统计图表） |

### 基础设施与运维技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **Docker** | 24+ | 容器化运行时 |
| **Docker Compose** | v2.20+ | 多容器编排（开发/测试/生产） |
| **Nginx** | 1.24+ | 反向代理 + 静态资源 + SSL 终结 |
| **Prometheus** | latest | 指标采集与时间序列数据库 |
| **Grafana** | latest | 可视化监控仪表盘 |
| **Jaeger** | latest | 分布式链路追踪 UI |
| **Let's Encrypt** | Certbot | 免费 HTTPS 证书 |

---

## 8 大微服务详解

### 服务架构总览图

```
                         ┌──────────────────────────────────────┐
                         │         用户浏览器 / 移动 App        │
                         └──────────────┬───────────────────────┘
                                        │ HTTPS
                         ┌──────────────▼───────────────────────┐
                         │     Nginx (SSL终结 + 静态资源)       │
                         │   - Vue.js 前端构建产物托管           │
                         │   - /api/* 转发到 Gateway             │
                         └──────────────┬───────────────────────┘
                                        │ HTTP :8080
                         ┌──────────────▼───────────────────────┐
                         │      API Gateway (YARP)               │
                         │  ┌─────────────────────────────────┐  │
                         │  │  路由转发:                        │  │
                         │  │  /api/identity/* → identity-svc │  │
                         │  │  /api/products/* → product-svc   │  │
                         │  │  /api/orders/*    → order-svc     │  │
                         │  │  /api/cart/*      → cart-svc      │  │
                         │  │  /api/payments/* → payment-svc    │  │
                         │  │  /api/inventory/*→ inventory-svc  │  │
                         │  │  /api/notify/*   → notify-svc     │  │
                         │  ├─────────────────────────────────┤  │
                         │  │  JWT 认证 (RSA 公钥验证)          │  │
                         │  │  限流 (Token Bucket: 100 req/min) │  │
                         │  │  CORS 白名单                      │  │
                         │  │  请求/响应日志                     │  │
                         │  │  API 聚合 (组合多个服务响应)       │  │
                         │  └─────────────────────────────────┘  │
                         └──────────────┬───────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
         ┌──────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
         │  Identity Service │ │Product Service│ │ Order Service  │
         │  (用户认证/权限)   │ │(商品/分类/搜索)│ │(订单/Saga)    │
         ├──────────────────┤ ├──────────────┤ ├───────────────┤
         │ PostgreSQL:16    │ │ PostgreSQL:16│ │ PostgreSQL:16  │
         │ Redis (会话/码)  │ │ Redis (缓存) │ │ Redis (状态)   │
         └──────────┬────────┘ └──────┬───────┘ └───────┬───────┘
                    │                 │                  │
         ┌──────────▼────────┐ ┌──────▼───────┐ ┌───────▼───────┐
         │   Cart Service    │ │Payment Svc   │ │ Inventory Svc  │
         │  (购物车/游客合并) │ │(多渠道支付)  │ │(库存/并发控制) │
         ├──────────────────┤ ├──────────────┤ ├───────────────┤
         │ Redis (Hash 存储) │ │ PostgreSQL   │ │ PostgreSQL     │
         └──────────┬────────┘ │ Redis (幂等) │ │ Redis (锁/缓存)│
                    │          └──────┬───────┘ └───────┬───────┘
                    │                 │                  │
                    └─────────────────┼──────────────────┘
                                      │
                         ┌────────────▼────────────┐
                         │    Notification Service   │
                         │  (邮件/短信/站内信)       │
                         ├───────────────────────────┤
                         │ RabbitMQ Consumer        │
                         │ MailKit (SMTP)            │
                         │ SMS Provider SDK          │
                         └───────────────────────────┘

              ┌──────────────────────────────────────────┐
              │        RabbitMQ (消息队列集群)             │
              │  ┌────────────────────────────────────┐   │
              │  │ Exchanges:                          │   │
              │  │  - cloudmall.direct (直连交换机)    │   │
              │  │  - cloudmall.topic  (主题交换机)    │   │
              │  │  - cloudmall.fanout (广播交换机)    │   │
              │  │                                     │   │
              │  │ Queues (关键队列):                   │   │
              │  │  - order.created                    │   │
              │  │  - order.payment.completed          │   │
              │  │  - order.payment.failed             │   │
              │  │  - inventory.reserved               │   │
              │  │  - inventory.released               │   │
              │  │  - cart.cleared                    │   │
              │  │  - notification.order              │   │
              │  │  - notification.payment            │   │
              │  └────────────────────────────────────┘   │
              └──────────────────────────────────────────┘

              ┌──────────────────────────────────────────┐
              │  基础设施支撑                             │
              │  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
              │  │PostgreSQL│ │  Redis   │ │ MinIO    │ │
              │  │ ×4 实例  │ │ Cluster  │ │ (图片)   │ │
              │  └──────────┘ └──────────┘ └──────────┘ │
              │  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
              │  │Elasticse-│ │Prometheus│ │ Grafana  │ │
              │  │ arch     │ │ + Jaeger │ │ Dashboard│ │
              │  └──────────┘ └──────────┘ └──────────┘ │
              └──────────────────────────────────────────┘
```

---

### 1️⃣ API Gateway（API 网关服务）

**端口**：8000（内部）/ 8080（Nginx 映射）
**职责**：系统的统一入口点，所有外部请求必须经过网关

#### 核心功能

##### 1.1 YARP 反向代理路由
```csharp
// appsettings.json 中的路由配置
"ReverseProxy": {
  "Routes": {
    "identity-route": {
      "ClusterId": "identity-cluster",
      "Match": { "Path": "/api/identity/{**catch-all}" }
    },
    "products-route": {
      "ClusterId": "products-cluster",
      "Match": { "Path": "/api/products/{**catch-all}" }
    },
    "orders-route": {
      "ClusterId": "orders-cluster",
      "Match": { "Path": "/api/orders/{**catch-all}" }
    },
    "cart-route": {
      "ClusterId": "cart-cluster",
      "Match": { "Path": "/api/cart/{**catch-all}" }
    },
    "payments-route": {
      "ClusterId": "payments-cluster",
      "Match": { "Path": "/api/payments/{**catch-all}" }
    },
    "inventory-route": {
      "ClusterId": "inventory-cluster",
      "Match": { "Path": "/api/inventory/{**catch-all}" }
    },
    "notifications-route": {
      "ClusterId": "notifications-cluster",
      "Match": { "Path": "/api/notifications/{**catch-all}" }
    }
  },
  "Clusters": {
    "identity-cluster": { "Destinations": { "destination1": { "Address": "http://identity-service:8001/" } } },
    "products-cluster": { "Destinations": { "destination1": { "Address": "http://product-service:8002/" } } },
    "orders-cluster": { "Destinations": { "destination1": { "Address": "http://order-service:8003/" } } },
    "cart-cluster": { "Destinations": { "destination1": { "Address": "http://cart-service:8004/" } } },
    "payments-cluster": { "Destinations": { "destination1": { "Address": "http://payment-service:8005/" } } },
    "inventory-cluster": { "Destinations": { "destination1": { "Address": "http://inventory-service:8006/" } } },
    "notifications-cluster": { "Destinations": { "destination1": { "Address": "http://notification-service:8007/" } } }
  }
}
```

##### 1.2 JWT RSA 公钥验证（无状态认证）
```csharp
// 网关只负责验证 Token 签名（不查询数据库）
// 使用 RSA 公钥验证，私钥仅保存在 Identity Service
public class JwtAuthenticationMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IConfiguration _config;

    public async Task InvokeAsync(HttpContext context)
    {
        var token = ExtractToken(context.Request);
        
        if (!string.IsNullOrEmpty(token))
        {
            var principal = ValidateToken(token);  // RSA 公钥验证
            if (principal != null)
            {
                context.User = principal;  // 设置用户身份（传递给下游服务）
                
                // 将关键 Claims 添加到 Header（下游服务可直接读取，无需再次验证）
                context.Request.Headers["X-User-Id"] = principal.FindFirst("sub")?.Value;
                context.Request.Headers["X-User-Role"] = principal.FindFirst(ClaimTypes.Role)?.Value;
                context.Request.Headers["X-User-Email"] = principal.FindFirst("email")?.Value;
            }
        }

        await _next(context);
    }
}
```

##### 1.3 速率限制（Token Bucket Algorithm）
```csharp
// 全局限流：每 IP 每分钟 100 个请求
app.UseRateLimiter(rateLimiterOptions =>
{
    rateLimiterOptions.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, IPAddress>(context =>
    {
        var ipAddress = context.Connection.RemoteIpAddress!;
        return RateLimitPartition.GetTokenBucketLimiter(ipAddress, _ =>
            new TokenBucketRateLimitOptions
            {
                TokenLimit = 100,           // 桶容量（突发允许）
                QueueProcessingOrder = QueueProcessingOrder.OldestFirst,
                QueueLimit = 50,            // 等待队列长度
                ReplenishmentPeriod = TimeSpan.FromSeconds(6),  // 补充间隔
                TokensPerPeriod = 10,       // 每次补充令牌数（= 100/分钟）
                AutoReplenish = true
            });
    });
    
    // 特殊端点更严格限制
    rateLimiterOptions.AddPolicy("auth-policy", context =>
        RateLimitPartition.GetTokenBucketLimiter(context.Connection.RemoteIpAddress!, _ =>
            new TokenBucketRateLimitOptions
            {
                TokenLimit = 10,
                ReplenishmentPeriod = TimeSpan.FromSeconds(60),
                TokensPerPeriod = 10,
                QueueLimit = 0  // 登录注册不允许排队
            }));
});
```

##### 1.4 API 聚合（BFF Pattern - Backend for Frontend）
```csharp
// 示例：首页聚合 API（调用多个服务组合数据）
[HttpGet("api/home/dashboard")]
public async Task<IActionResult> GetDashboard()
{
    var userId = GetUserFromContext();
    
    // 并行调用多个下游服务
    var tasks = new[]
    {
        _productClient.GetFeaturedProductsAsync(),      // 推荐商品
        _productClient.GetCategoriesAsync(),            // 分类列表
        _cartClient.GetCartSummaryAsync(userId),        // 购物车摘要
        _orderClient.GetRecentOrdersAsync(userId, 3),   // 最近订单
        _notificationClient.GetUnreadCountAsync(userId) // 未读通知数
    };
    
    await Task.WhenAll(tasks);
    
    return Ok(new HomeDashboardDto
    {
        FeaturedProducts = tasks[0].Result,
        Categories = tasks[1].Result,
        CartSummary = tasks[2].Result,
        RecentOrders = tasks[3].Result,
        UnreadNotifications = tasks[4].Result
    });
}
```

##### 1.5 请求日志与链路追踪注入
- 每个请求生成 TraceId（透传到所有下游服务）
- OpenTelemetry 集成（自动上报 Span 到 Jaeger）
- 请求/响应耗时记录（慢请求 > 1s 告警）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 转发目标 |
|------|------|------|---------|
| GET | `/api/health` | 网关健康检查 | 本地 |
| GET | `/api/home/dashboard` | 首页聚合数据 | 聚合多服务 |
| * | `/api/identity/*` | 用户认证相关 | identity-service:8001 |
| * | `/api/products/*` | 商品相关 | product-service:8002 |
| * | `/api/orders/*` | 订单相关 | order-service:8003 |
| * | `/api/cart/*` | 购物车相关 | cart-service:8004 |
| * | `/api/payments/*` | 支付相关 | payment-service:8005 |
| * | `/api/inventory/*` | 库存相关 | inventory-service:8006 |
| * | `/api/notifications/*` | 通知相关 | notification-service:8007 |

---

### 2️⃣ Identity Service（用户认证与权限服务）

**端口**：8001
**数据库**：PostgreSQL (cloudmall_identity)
**缓存**：Redis（会话存储、验证码、Token 黑名单）

#### 核心功能

##### 2.1 用户注册与登录
- **注册**：邮箱/手机号 + 密码（bcrypt 哈希）、邮箱验证码验证（可选）
- **登录**：支持用户名/邮箱/手机号三字段登录
- **JWT 生成**：使用 RSA 私钥签署（2048位或4096位），Access Token 30分钟 + Refresh Token 7天
- **OAuth2 第三方登录**：
  - GitHub OAuth2（开发者群体）
  - Google OAuth2（国际用户）
  - 微信 OAuth2（国内用户，需 ICP 备案）

##### 2.2 RBAC 权限模型（基于角色的访问控制）
```
角色层次结构：
├── SuperAdmin（超级管理员）
│   ├── 系统配置管理
│   ├── 服务监控
│   └── 数据导出
├── Admin（管理员）
│   ├── 商品管理（CRUD）
│   ├── 订单管理（查看/处理退款）
│   ├── 用户管理（禁用/角色分配）
│   └── 库存管理
├── Seller（商家/运营）
│   ├── 商品发布与管理
│   ├── 订单处理
│   └── 销售数据查看
└── User（普通买家）
    ├── 浏览商品
    ├── 下单购买
    ├── 个人中心
    └── 收货地址管理
```

**权限检查实现**：
```csharp
// 自定义 Authorize Attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class RequireRoleAttribute : AuthorizeAttribute
{
    public RequireRoleAttribute(params string[] roles) : base()
    {
        Roles = string.Join(",", roles);
    }
}

// Controller 使用
[RequireRole("Admin", "Seller")]
[HttpPost("products")]
public async Task<IActionResult> CreateProduct([FromBody] CreateProductRequest request)
{
    // 只有 Admin 或 Seller 角色可以创建商品
    ...
}

// 或者使用 Policy（更灵活的策略授权）
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ProductManagement", policy =>
        policy.RequireRole("Admin", "Seller"));
    options.AddPolicy("OrderManagement", policy =>
        policy.RequireRole("Admin", "Seller")
              .RequireClaim("department", "sales"));  // 额外 Claim 要求
});
```

##### 2.3 手机验证码登录/注册
- 发送 SMS 验证码（阿里云短信 / 腾讯云短信 SDK）
- 验证码存储到 Redis（Key: `sms:code:{phone}`, TTL: 5分钟）
- 频率限制：同一手机号 60秒内只能发送1次，每天最多10次
- 验证码校验：一次性使用，验证后立即删除

##### 2.4 刷新 Token 与吊销
- Refresh Token 存储到 PostgreSQL 表（支持跨实例共享）
- 支持多设备登录（每设备独立 Refresh Token）
- 强制下线指定设备（删除该设备的 Refresh Token）
- 全局注销（删除用户所有 Refresh Token）

**数据库表**：
- `users` — 用户基本信息
- `user_roles` — 用户角色关联（多对多）
- `user_logins` — 第三方登录记录（GitHub/Google/WeChat）
- `refresh_tokens` — Refresh Token 存储
- `user_addresses` — 收货地址
- `verification_codes` — 验证码记录（审计用）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/auth/register` | 邮箱密码注册 | 公开 |
| POST | `/api/auth/login` | 密码登录 | 公开 |
| POST | `/api/auth/sms-login` | 手机验证码登录 | 公开 |
| POST | `/api/auth/sms-send` | 发送验证码 | 公开 |
| POST | `/api/auth/refresh` | 刷新 Token | 公开（Cookie） |
| POST | `/api/auth/logout` | 登出 | 已认证 |
| GET | `/api/auth/github` | GitHub OAuth2 重定向 | 公开 |
| GET | `/api/auth/github-callback` | GitHub 回调 | 公开 |
| PUT | `/api/auth/password` | 修改密码 | 已认证 |
| PUT | `/api/auth/profile` | 更新资料 | 已认证 |
| GET | `/api/users/me` | 当前用户信息 | 已认证 |
| GET | `/api/users/{id}` | 用户公开信息 | 公开 |
| CRUD | `/api/addresses` | 收货地址管理 | 已认证 |
| POST | `/api/admin/users/{id}/roles` | 分配角色（管理员） | Admin |

---

### 3️⃣ Product Service（商品服务）

**端口**：8002
**数据库**：PostgreSQL (cloudmall_products)
**缓存**：Redis（热门商品/分类/推荐）
**搜索**：Elasticsearch (cloudmall_products_index)
**存储**：MinIO（商品图片 S3 兼容）

#### 核心功能

##### 3.1 商品 SPU/SKU 数据模型
```
SPU (Standard Product Unit) — 标准产品单元
├── 基本信息：名称、描述、品牌、分类ID、主图
├── 销售属性模板：颜色、尺寸、材质等（由分类决定）
└── SKU 列表：具体规格组合

SKU (Stock Keeping Unit) — 库存保持单元
├── SPU ID（所属商品）
├── 规格属性：{"颜色":"红色","尺寸":"XL"}
├── 价格（不同规格可能不同价格）
├── 库存数量（每个 SKU 独立库存）
├── 条形码/编码
└── 状态（上架/下架/缺货）
```

**实体关系**：
```
Category (分类树) 1:N → SPU (商品) 1:N → SKU (规格)
Brand (品牌)     1:N → SPU
SPU N:M → Tag (标签，多对多)
SPU 1:N → ProductImage (图片集)
SPU 1:N → ProductAttribute (销售属性定义)
```

##### 3.2 分类树管理
- 支持无限层级分类（通常 3 级足够：一级分类 > 二级分类 > 三级分类）
- 使用 Closure Table（物化路径）模式优化树查询
- 分类面包屑导航生成算法
- 分类下的商品计数（实时或定时更新）

##### 3.3 Elasticsearch 全文搜索
- **索引字段**：名称（权重 5x）、描述（权重 2x）、品牌（权重 3x）、分类名（权重 2x）、标签（权重 3x）、SKU 属性值
- **搜索功能**：关键词匹配 + 分类筛选 + 价格区间 + 品牌筛选 + 排序（相关性/价格升序/价格降序/销量/新品）
- **自动补全**：Completion Suggester（输入即提示）
- **同义词扩展**："手机" ↔ "移动电话"、"笔记本" ↔ "laptop"
- **索引同步策略**：商品 CRUD 时异步同步到 ES（通过 MassTransit 发布事件 → Search Worker 消费并更新索引）

##### 3.4 商品推荐系统（基础版）
- **基于内容的推荐**：根据用户浏览/购买历史中的商品标签，推荐相似标签的商品
- **协同过滤（简化版）："买了还买"——购买某商品的用户也购买了其他哪些商品（基于 OrderItem 统计）
- **热门推荐**：按销量/浏览量排序的 Top N 商品（Redis Sorted Set 缓存，每小时刷新）
- **新品推荐**：最近 7 天上架的商品

##### 3.5 图片管理
- MinIO S3 兼容存储（主图 + 详情图 + SKU 规格图）
- 自动生成缩略图（200x200 小图、400x400 中图、800x800 大图）
- WebP 格式转换（减小体积 25-80%）
- CDN URL 返回（通过 Nginx 代理 MinIO）

**数据库表**：
- `categories` / `category_paths`（Closure Table）
- `brands`
- `products` (SPU)
- `product_skus` (SKU)
- `product_images`
- `product_attributes` / `product_attribute_values`
- `tags` / `product_tags`（多对多中间表）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/products` | 商品列表（分页/筛选/排序） | 公开 |
| GET | `/api/products/{id}` | 商品详情（含 SKU 列表） | 公开 |
| GET | `/api/products/slug/{slug}` | 按 Slug 获取详情 | 公开 |
| POST | `/api/products` | 创建商品（SPU + SKU） | Seller/Admin |
| PUT | `/api/products/{id}` | 更新商品 | Seller/Admin |
| DELETE | `/api/products/{id}` | 下架/删除商品 | Admin |
| PUT | `/api/products/{id}/status` | 上架/下架切换 | Seller/Admin |
| GET | `/api/products/search` | Elasticsearch 全文搜索 | 公开 |
| GET | `/api/products/suggest` | 搜索建议/自动补全 | 公开 |
| GET | `/api/products/recommended` | 个性化推荐 | 已认证 |
| GET | `/api/products/hot` | 热门商品 Top N | 公开 |
| GET | `/api/products/new` | 新品上架 | 公开 |
| CRUD | `/api/categories` | 分类管理（CRUD + 树形） | Admin |
| CRUD | `/api/brands` | 品牌管理 | Admin |
| CRUD | `/api/tags` | 标签管理 | Seller/Admin |
| POST | `/api/products/{id}/images` | 上传商品图片 | Seller/Admin |

---

### 4️⃣ Order Service（订单服务）— 核心！

**端口**：8003
**数据库**：PostgreSQL (cloudmall_orders)
**缓存**：Redis（订单状态缓存、幂等键）
**消息**：RabbitMQ Producer（发布订单事件）

#### 核心功能

##### 4.1 下单 Saga 流程（分布式事务）

这是整个系统最核心的业务流程，涉及 4 个服务的协作：

```
用户点击"提交订单"
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│  Order Service: 创建订单（Pending 状态）                       │
│  - 生成订单号（时间戳 + 随机数：202604171234567890ABCD）       │
│  - 从 Cart Service 获取购物车数据                              │
│  - 计算金额（商品总价 + 运费 - 优惠折扣）                       │
│  - 选择收货地址                                               │
│  - 保存订单到数据库                                           │
│  - 发布事件: OrderCreated                                    │
└──────────────────────┬──────────────────────────────────────┘
                       │ RabbitMQ: order.created
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Inventory Service — 库存锁定                        │
│  - 接收 OrderCreated 事件                                   │
│  - 遍历订单中每个 SKU                                        │
│  - 检查库存是否充足                                          │
│  - 扣减库存（预留库存，非真实扣减）                            │
│  - 成功: 发布 InventoryReserved 事件                         │
│  - 失败: 发布 InventoryReservationFailed 事件               │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          │ 成功                     │ 失败
          ▼                         ▼
┌──────────────────┐    ┌──────────────────────────────┐
│ Step 2: 清空购物车 │    │ Compensation: 取消订单        │
│ Cart Service      │    │ Order Status → Cancelled     │
│ - 删除购物车商品   │    │ 发布: OrderCancelled         │
│ - 发布:CartCleared│    └──────────────────────────────┘
└──────────┬────────┘
           │ RabbitMQ: cart.cleared
           ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Payment Service — 创建支付单                       │
│  - 生成支付单号                                             │
│  - 调用第三方支付接口创建预支付交易                           │
│  - 成功: 等待用户付款                                       │
│  - 用户付款成功回调 → PaymentCompleted 事件                │
│  - 用户付款失败/超时 → PaymentFailed / PaymentExpired 事件  │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          │ 支付成功                 │ 支付失败/超时
          ▼                         ▼
┌──────────────────┐    ┌──────────────────────────────────┐
│ Step 4: 确认订单  │    │ Compensation Chain:              │
│ Order Status →    │    │ 1. 释放库存 (Inventory Release)  │
│   Paid            │    │ 2. 退款处理 (Payment Refund)      │
│ 发布: OrderPaid   │    │ 3. 订单 Status → Expired/Cancelled│
│ 通知用户 + 商家    │    └──────────────────────────────────┘
└──────────────────┘
```

##### 4.2 Saga 编排器实现（Process Manager 模式）

```csharp
/// <summary>
/// 下单 Saga 编排器（使用 MassTransit + RabbitMQ 的 State Machine）
/// </summary>
public class OrderSagaStateMachine :
    MassTransitStateMachine<OrderSagaState>
{
    public OrderSagaStateMachine()
    {
        // 定义状态
        Event(() => OrderCreated, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => InventoryReserved, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => InventoryFailed, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => CartCleared, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => PaymentCompleted, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => PaymentFailed, x => x.CorrelateById(m => m.Message.OrderId));

        // 定义状态流转
        Initially(
            When(OrderCreated)
                .Then(ctx => {
                    ctx.Saga.CreatedAt = DateTime.UtcNow;
                    ctx.Saga.CurrentStep = "InventoryReserve";
                    Publish(ctx => new ReserveInventoryCommand(...));  // Step 1: 锁库存
                })
                .TransitionTo(ReservingInventory)
        );

        During(ReservingInventory,
            When(InventoryReserved)
                .Then(ctx => {
                    ctx.Saga.CurrentStep = "ClearCart";
                    Publish(ctx => new ClearCartCommand(...));  // Step 2: 清购物车
                })
                .TransitionTo(ClearingCart),
            
            When(InventoryFailed)
                .Then(ctx => {
                    ctx.Saga.Status = "Cancelled";
                    Publish(ctx => new CancelOrderCommand(...));  // 补偿: 取消订单
                })
                .TransitionTo(Failed)
        );

        During(ClearingCart,
            When(CartCleared)
                .Then(ctx => {
                    ctx.Saga.CurrentStep = "ProcessPayment";
                    Publish(ctx => new CreatePaymentCommand(...));  // Step 3: 创建支付
                })
                .TransitionTo(AwaitingPayment)
        );

        During(AwaitingPayment,
            When(PaymentCompleted)
                .Then(ctx => {
                    ctx.Saga.Status = "Paid";
                    ctx.Saga.CompletedAt = DateTime.UtcNow;
                    Publish(ctx => new ConfirmOrderCommand(...));  // Step 4: 确认订单
                })
                .TransitionTo(Completed),

            When(PaymentFailed)
                .Then(ctx => StartCompensation(ctx))  // 启动补偿事务
                .TransitionTo(Compensating)
        );
    }

    private void StartPaymentCompensation(BehaviorContext<OrderSagaState, PaymentFailedEvent> ctx)
    {
        // 补偿顺序（与执行顺序相反）：释放库存 → 退款
        Publish(ctx => new ReleaseInventoryCommand(...));
        Publish(ctx => new ProcessRefundCommand(...));
    }
}
```

##### 4.3 订单状态机（10 种状态）

```
                    ┌──────────────┐
                    │   Pending    │ ← 创建订单初始状态
                    │  （待支付）   │
                    └──────┬───────┘
                           │ 支付成功
                           ▼
                    ┌──────────────┐
              ┌─────▶│    Paid      │◀──────┐
              │     │  （已支付）   │       │
              │     └──────┬───────┘       │
              │            │               │
              │  卖家发货    │ 支付超时(30min)│
              │            ▼               │
              │     ┌──────────────┐       │
              │     │  Shipped     │       │
              │     │  （已发货）   │       │
              │     └──────┬───────┘       │
              │            │               │
              │  用户确认收货 │  用户取消    │
              │            ▼               │
              │     ┌──────────────┐       │
              │     │ Completed    │       │
              │     │  （已完成）   │       │
              │     └──────────────┘       │
              │                            │
              │  其他状态路径：              │
              │                            │
              │  Pending ──(用户取消)──▶ Cancelled（已取消）     │
              │  Pending ──(系统取消)──▶ Expired（已过期）       │
              │  Paid ──(申请退款)───▶ Refunding（退款中）       │
              │  Refund ──(退款完成)──▶ Refunded（已退款）       │
              │  Shipped ──(售后申请)──▶ AfterSale（售后中）     │
              └─────────────────────────────────────────────────┘

状态枚举定义：
public enum OrderStatus
{
    Pending = 0,       // 待支付（刚创建）
    Paid = 1,          // 已支付（等待发货）
    Shipped = 2,       // 已发货（物流途中）
    Completed = 3,     // 已完成（确认收货）
    Cancelled = 4,     // 已取消（用户主动或库存不足）
    Expired = 5,       // 已过期（超时未支付，默认30分钟）
    Refunding = 6,     // 退款中
    Refunded = 7,      // 已退款
    AfterSale = 8,     // 售后处理中
    Closed = 9         // 已关闭（交易结束，不可操作）
}
```

##### 4.4 幂等性保证
- 每个写操作使用 Idempotency Key（客户端生成的 UUID）
- Redis 存储已处理的 Key（TTL = 24h）
- 重复请求返回相同结果而非重复执行

##### 4.5 订单超时自动取消（Worker Service）
- BackgroundService 定时扫描 Pending 状态超过 30 分钟的订单
- 触发补偿流程：释放库存 + 取消订单 + 通知用户

**数据库表**：
- `orders` — 订单主表
- `order_items` — 订单项（商品明细）
- `order_addresses` — 收货地址快照
- `order_status_history` — 状态变更日志（审计追踪）
- `order_payments` — 支付记录
- `order_refunds` — 退款记录
- `order_sagas` — Saga 状态持久化（用于故障恢复）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/orders` | 创建订单（触发 Saga） | 已认证 |
| GET | `/api/orders` | 我的订单列表（分页/筛选） | 已认证 |
| GET | `/api/orders/{id}` | 订单详情 | 本人/管理员 |
| PUT | `/api/orders/{id}/cancel` | 取消订单（待支付状态） | 本人 |
| PUT | `/api/orders/{id}/confirm` | 确认收货（已发货状态） | 本人 |
| POST | `/api/orders/{id}/refund` | 申请退款 | 本人 |
| GET | `/api/orders/{id}/timeline` | 订单状态时间线 | 本人 |
| GET | `/api/orders/tracking/{id}` | 物流跟踪信息 | 本人 |
| CRUD | `/api/admin/orders` | 管理员订单管理 | Admin |

---

### 5️⃣ Cart Service（购物车服务）

**端口**：8004
**存储**：Redis Hash（主存储） + PostgreSQL（持久化备份，可选）

#### 核心功能

##### 5.1 Redis Hash 购物车数据结构
```
Key:   cart:{userId}          # 登录用户购物车
       cart:guest:{guestId}    # 游客购物车（临时标识）

Field: {skuId}                # Hash Field = SKU ID
Value: JSON String:
{
  "skuId": "sku_001",
  "productId": "prod_042",
  "productName": "iPhone 15 Pro",
  "productImage": "https://cdn.../iphone15.jpg",
  "skuAttributes": {"颜色": "原色钛金属", "存储": "256GB"},
  "price": 8999.00,
  "quantity": 2,
  "selected": true,           // 是否选中（用于结算）
  "addedAt": "2026-04-17T10:30:00Z"
}

Redis Commands:
HGETALL cart:user_abc123        # 获取全部购物车项
HGET cart:user_abc123 sku_001   # 获取单个商品
HSET cart:user_abc123 sku_001 '{json}'  # 添加/更新
HDEL cart:user_abc123 sku_001   # 删除商品
EXPIRE cart:guest_xyz789 604800 # 游客购物车 7 天过期
```

##### 5.2 购物车原子操作
```csharp
// 添加到购物车（原子操作：不存在则创建，存在则累加数量）
public async Task AddToCartAsync(string userId, CartItemDto item)
{
    var db = _redis.GetDatabase();
    
    // Lua 脚本保证原子性（检查库存上限 + 数量累加）
    var script = @"
        local current = redis.call('HGET', KEYS[1], ARGV[1])
        if current then
            local data = cjson.decode(current)
            data.quantity = math.min(data.quantity + tonumber(ARGV[2]), 99)
            redis.call('HSET', KEYS[1], ARGV[1], cjson.encode(data))
            return data.quantity
        else
            redis.call('HSET', KEYS[1], ARGV[1], ARGV[3])
            return tonumber(ARGV[2])
        end
    ";
    
    var result = await db.ScriptEvaluateAsync(
        ScriptCache.GetScriptHash(script),
        new[] { RedisKey.Cart(userId) },  // KEYS[1]
        new RedisValue[] { item.SkuId, item.Quantity, JsonSerializer.Serialize(item) }
    );
    
    // 发布购物车变更事件（可选：用于实时同步前端）
    await _bus.Publish(new CartUpdatedEvent { UserId = userId });
}
```

##### 5.3 游客购物车合并（登录时）
```csharp
public async Task MergeGuestCartAsync(string guestId, string userId)
{
    // 1. 获取游客购物车
    var guestItems = await GetAllItemsAsync(guestId);
    if (!guestItems.Any()) return;
    
    // 2. 遍历游客购物车每一项，合并到用户购物车
    foreach (var guestItem in guestItems)
    {
        var existingItem = await GetItemAsync(userId, guestItem.SkuId);
        if (existingItem != null)
        {
            // 已存在：累加数量（最大 99）
            var newQty = Math.Min(existingItem.Quantity + guestItem.Quantity, 99);
            await UpdateQuantityAsync(userId, guestItem.SkuId, newQty);
        }
        else
        {
            // 不存在：直接添加
            guestItem.UserId = userId;
            await AddToCartAsync(userId, guestItem);
        }
    }
    
    // 3. 删除游客购物车
    await _redis.GetDatabase().KeyDeleteAsync(RedisKey.Cart(guestId));
    
    _logger.LogInformation("游客购物车已合并: GuestId={GuestId}, UserId={UserId}, Items={Count}",
        guestId, userId, guestItems.Count);
}
```

##### 5.4 购物车结算（为订单服务提供数据）
- 返回选中的购物车项（selected=true）
- 实时计算总价（从 Redis 读取最新价格，或从 Product Service 获取实时价格）
- 校验 SKU 是否仍然有效（未下架、有库存）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/cart` | 获取购物车（全部商品） | 已认证/游客 |
| POST | `/api/cart/items` | 添加商品到购物车 | 已认证/游客 |
| PUT | `/api/cart/items/{skuId}` | 更新数量 | 已认证/游客 |
| DELETE | `/api/cart/items/{skuId}` | 删除商品 | 已认证/游客 |
| PUT | `/api/cart/items/{skuId}/select` | 选中/取消选中 | 已认证/游客 |
| PUT | `/api/cart/select-all` | 全选/取消全选 | 已认证/游客 |
| DELETE | `/api/cart/clear` | 清空购物车 | 已认证/游客 |
| GET | `/api/cart/summary` | 购物车摘要（总件数/总价） | 已认证/游客 |
| GET | `/api/cart/checkout` | 结算数据（选中商品+价格校验） | 已认证 |
| POST | `/api/cart/merge` | 合并游客购物车（登录时调用） | 已认证 |
| GET | `/api/cart/count` | 购物车商品数量（Header Badge 用） | 已认证/游客 |

---

### 6️⃣ Payment Service（支付服务）

**端口**：8005
**数据库**：PostgreSQL (cloudmall_payments)
**缓存**：Redis（幂等键、支付状态缓存）

#### 核心功能

##### 6.1 多渠道支付支持
- **支付宝**（Alipay SDK）：扫码支付 / APP 支付 / H5 支付 / 网页支付
- **微信支付**（WeChatPay SDK）：Native / JSAPI / H5 / APP
- **模拟支付**（开发环境）：自动成功/手动确认（用于测试）
- **余额支付**（可选）：平台内部钱包余额

##### 6.2 支付流程
```
1. Order Service 发布 CreatePaymentCommand
       ↓
2. Payment Service 创建支付记录（Status=Created）
       ↓
3. 调用第三方支付接口获取预支付参数
   - 支付宝: alipay.trade.precreate (返回二维码URL)
   - 微信: unifiedorder (返回 prepay_id)
       ↓
4. 返回给前端：支付参数（二维码/调起SDK参数）
       ↓
5. 用户扫码/确认支付
       ↓
6. 第三方平台异步回调通知（Webhook/Notify URL）
   - 支付宝: /api/payments/callback/alipay
   - 微信: /api/payments/callback/wechat
       ↓
7. Payment Service 验签（防止伪造回调）
   - 检查签名正确性
   - 检查金额是否匹配
   - 检查订单状态（幂等性：已支付则直接返回成功）
       ↓
8. 更新支付状态为 Paid
       ↓
9. 发布 PaymentCompleted 事件（触发 Order Service 确认订单）
       ↓
10. 返回 success 给第三方平台（否则平台会重复推送）
```

##### 6.3 回调验签（安全性关键）
```csharp
[HttpPost("callback/alipay")]
public async Task<IActionResult> AlipayCallback([FromForm] IDictionary<string, string> form)
{
    _logger.LogInformation("收到支付宝回调: TradeNo={TradeNo}, TradeStatus={TradeStatus}",
        form.GetValue("trade_no"), form.GetValue("trade_status"));

    try
    {
        // 1. 验签（最关键！防止伪造回调）
        var signChecked = await _alipayService.VerifySignAsync(form);
        if (!signChecked)
        {
            _logger.LogWarning("支付宝回调验签失败！可能是伪造请求");
            return Ok("failure");  // 必须返回 failure
        }

        // 2. 检查交易状态
        var tradeStatus = form["trade_status"];
        if (tradeStatus != "TRADE_SUCCESS" && tradeStatus != "TRADE_FINISHED")
        {
            return Ok("success");  // 交易未完成但验签通过，返回 success 避免重复推送
        }

        // 3. 获取支付参数
        var outTradeNo = form["out_trade_no"];      // 我们的支付单号
        var tradeNo = form["trade_no"];             // 平台交易号
        var totalAmount = decimal.Parse(form["total_amount"]);

        // 4. 幂等性检查（防止重复处理）
        var existingPayment = await _paymentRepository.GetByOutTradeNoAsync(outTradeNo);
        if (existingPayment?.Status == PaymentStatus.Paid)
        {
            _logger.LogInformation("支付单已处理，跳过: OutTradeNo={OutTradeNo}", outTradeNo);
            return Ok("success");
        }

        // 5. 更新支付状态
        await _paymentService.CompletePaymentAsync(outTradeNo, tradeNo, totalAmount);

        // 6. 发布支付完成事件（触发后续流程）
        await _eventBus.PublishAsync(new PaymentCompletedEvent
        {
            PaymentId = existingPayment!.Id,
            OrderId = existingPayment.OrderId,
            TransactionId = tradeNo,
            Amount = totalAmount,
            PaidAt = DateTime.UtcNow
        });

        return Ok("success");  // 必须返回纯文本 "success"
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "处理支付宝回调异常");
        return Ok("failure");
    }
}
```

##### 6.4 退款流程
- 申请退款 → 创建退款记录 → 调用第三方退款接口 → 异步回调确认 → 发布 RefundCompleted 事件
- 退款金额 ≤ 原支付金额
- 支持部分退款和全额退款
- 退款时效：支付宝即时到账 / 微信 1-3 工作日

**数据库表**：
- `payments` — 支付记录
- `payment_callbacks` — 回调记录（审计）
- `refunds` — 退款记录
- `refund_callbacks` — 退款回调记录

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/payments/create` | 创建支付单（内部服务调用） | 内部 |
| GET | `/api/payments/{id}` | 支付详情 | 本人/管理员 |
| POST | `/api/payments/callback/alipay` | 支付宝回调 | 公开（需 IP 白名单） |
| POST | `/api/payments/callback/wechat` | 微信支付回调 | 公开（需 IP 白名单） |
| POST | `/api/payments/{id}/refund` | 申请退款 | 本人 |
| GET | `/api/payments/{id}/refund-status` | 退款进度查询 | 本人 |
| POST | `/api/payments/simulate/pay` | 模拟支付（开发环境） | 开发者 |

---

### 7️⃣ Inventory Service（库存服务）

**端口**：8006
**数据库**：PostgreSQL (cloudmall_inventory)
**缓存**：Redis（库存热点数据 + 分布式锁）

#### 核心功能

##### 7.1 库存数据模型
```
Inventory (库存表):
- SkuId (PK, FK → Product_Service.product_skus)
- Quantity (当前可用库存)
- ReservedQuantity (已预留/锁定库存，Saga 中间态)
- SoldQuantity (已售出数量)
- Version (乐观锁版本号)
- LastUpdatedAt

库存关系:
可用库存 = Quantity - ReservedQuantity
总库存 = Quantity (入库时增加)
```

##### 7.2 三种并发控制方案

**方案 A：乐观锁（适合低竞争场景）**
```csharp
public async Task<bool> ReserveAsync(string skuId, int quantity)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        var inventory = await _context.Inventories
            .FirstOrDefaultAsync(i => i.SkuId == skuId);

        if (inventory == null || inventory.AvailableQuantity < quantity)
            return false;  // 库存不足

        inventory.ReservedQuantity += quantity;
        inventory.Version++;  // 乐观锁
        
        // Version 条件确保没有并发修改
        var affected = await _context.SaveChangesAsync();
        
        if (affected == 0)
            throw new DbUpdateConcurrencyException();  // 并发冲突
        
        await transaction.CommitAsync();
        return true;
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

**方案 B：悲观锁（SELECT FOR UPDATE，适合高竞争场景）**
```csharp
public async Task<bool> ReserveWithPessimisticLockAsync(string skuId, int quantity)
{
    using var transaction = await _context.Database.BeginTransactionAsync(IsolationLevel.Serializable);
    try
    {
        // SELECT ... FOR UPDATE 加行级排他锁
        var inventory = await _context.Inventories
            .FromSqlInterpolated($"""
                SELECT * FROM inventories 
                WHERE sku_id = {skuId} 
                FOR UPDATE  -- 加锁，其他事务必须等待
                """)
            .FirstOrDefaultAsync();

        if (inventory == null || inventory.AvailableQuantity < quantity)
            return false;

        inventory.ReservedQuantity += quantity;
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
        return true;
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

**方案 C：Redis 分布式锁 + Lua 脚本（最高性能）**
```csharp
// Redis 原子扣减（Lua 保证原子性）
public async Task<bool> ReserveWithRedisAsync(string skuId, int quantity)
{
    var db = _redis.GetDatabase();
    
    // Lua 脚本：原子检查并扣减
    var script = @"
        local key = KEYS[1]
        local reserveKey = KEYS[2]
        local qty = tonumber(ARGV[1])
        
        local available = redis.call('GET', key)
        if not available then
            return -1  -- 库存 Key 不存在
        end
        
        available = tonumber(available)
        if available < qty then
            return 0   -- 库存不足
        end
        
        -- 扣减可用库存
        redis.call('DECRBY', key, qty)
        -- 增加预留库存
        redis.call('INCRBY', reserveKey, qty)
        
        return 1   -- 成功
    ";
    
    var result = (long)await db.ScriptEvaluateAsync(
        script,
        new[] { $"inv:avail:{skuId}", $"inv:reserved:{skuId}" },
        new RedisValue[] { quantity });
    
    return result == 1;
}
```

**本项目采用方案**：正常流程使用 Redis Lua（高性能）；Redis 故障时降级为数据库乐观锁；管理员后台操作使用悲观锁（数据一致性优先）。

##### 7.3 库存释放（补偿操作）
- Saga 失败时调用：将 ReservedQuantity 还原回 Quantity
- 订单超时未支付自动释放（定时任务）
- 手动释放（管理员操作）

##### 7.4 库存预警
- 库存低于阈值（如 10 件）时发送预警通知给商家
- 缺货商品自动下架或标记为"缺货"
- 每日库存报表（销量 TOP 商品、库存周转率）

**数据库表**：
- `inventories` — 库存主表
- `inventory_transactions` — 库存变动流水（入库/出库/预留/释放/调整）
- `inventory_alerts` — 库存预警记录
- `stock_ins` — 入库记录

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/inventory/skus/{skuId}` | 查询 SKU 库存 | 公开 |
| GET | `/api/inventory/batch` | 批量查询库存（购物车用） | 内部服务 |
| POST | `/api/inventory/reserve` | 预留库存（Saga Step 1） | 内部服务 |
| POST | `/api/inventory/release` | 释放库存（补偿操作） | 内部服务 |
| POST | `/api/inventory/deduct` | 确认扣减（订单完成后） | 内部服务 |
| POST | `/api/inventory/restock` | 入库补货 | Seller/Admin |
| GET | `/api/inventory/alerts` | 库存预警列表 | Seller/Admin |
| GET | `/api/inventory/report` | 库存报表 | Seller/Admin |

---

### 8️⃣ Notification Service（通知服务）

**端口**：8007
**消息**：RabbitMQ Consumer（纯消费者，不暴露 HTTP API 或仅暴露管理接口）

#### 核心功能

##### 8.1 通知渠道
- **邮件通知**（MailKit SMTP）：订单确认、发货通知、退款通知、营销邮件
- **短信通知**（阿里云/腾讯云 SMS SDK）：验证码、订单状态变更、物流更新
- **站内信**（数据库存储 + WebSocket 推送）：系统公告、优惠活动、互动消息
- **Push 推送**（可选，极光推送/个推）：APP 端推送

##### 8.2 事件驱动的通知处理
```csharp
// MassTransit Consumer：监听所有业务事件并转换为通知
public class OrderNotificationConsumer : IConsumer<OrderCreatedEvent>,
                                         IConsumer<OrderPaidEvent>,
                                         IConsumer<OrderShippedEvent>,
                                         IConsumer<OrderCompletedEvent>,
                                         IConsumer<OrderCancelledEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        var orderEvent = context.Message;
        var user = await _userService.GetByIdAsync(orderEvent.UserId);
        
        // 1. 发送邮件：订单创建确认
        await _emailService.SendAsync(new EmailMessage
        {
            To = user.Email,
            Subject = $"订单创建成功 - 订单号 {orderEvent.OrderNumber}",
            Template = "order-created",
            Model = new { OrderNumber = orderEvent.OrderNumber, Items = orderEvent.Items, Total = orderEvent.TotalAmount }
        });
        
        // 2. 发送站内信
        await _internalMessageService.SendAsync(new InternalMessage
        {
            UserId = orderEvent.UserId,
            Title = "订单创建成功",
            Content = $"您的订单 {orderEvent.OrderNumber} 已创建，请及时支付。",
            Type = MessageType.Order
        });
        
        _logger.LogInformation("订单创建通知已发送: OrderId={OrderId}, UserId={UserId}", 
            orderEvent.OrderId, orderEvent.UserId);
    }

    // 类似地实现其他事件的处理...
}
```

##### 8.3 通知模板引擎
- 支持 Razor 模板（.cshtml）渲染邮件 HTML
- 变量替换：{{UserName}}、{{OrderNumber}}、{{ProductName}} 等
- 多语言模板（中文/英文）
- A/B 测试支持（不同用户看到不同的邮件文案）

##### 8.4 发送频率限制
- 同一用户同一类型通知：每小时最多 5 封邮件 / 10 条短信
- 营销类通知：每天最多 1 封（用户可在设置中关闭）
- 使用 Redis Sliding Window 计数器实现

**数据库表**（如果需要持久化通知记录）：
- `notifications` — 通知记录
- `notification_templates` — 通知模板
- `email_logs` — 邮件发送日志
- `sms_logs` — 短信发送日志
- `notification_preferences` — 用户通知偏好设置

**API 端点汇总**（主要是管理接口）：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/notifications` | 我的通知列表 | 已认证 |
| PUT | `/api/notifications/{id}/read` | 标记已读 | 已认证 |
| PUT | `/api/notifications/read-all` | 全部标记已读 | 已认证 |
| GET | `/api/notifications/unread-count` | 未读数量 | 已认证 |
| GET | `/api/notifications/settings` | 通知偏好设置 | 已认证 |
| PUT | `/api/notifications/settings` | 更新通知偏好 | 已认证 |
| POST | `/api/admin/notifications/send` | 手动发送通知（管理员） | Admin |
| GET | `/api/admin/notification-logs` | 发送日志查询 | Admin |

---

## 核心业务流程详解

### 1. 下单 Saga 完整流程（含补偿事务）

这是 CloudMall 最重要、最复杂的业务流程，涉及 4 个服务的协作：

```
时间线 →

[T0]    用户提交订单请求（POST /api/orders）
        │
        ▼
[T1]    Order Service:
        │  1. 参数验证（地址、购物车不为空）
        │  2. 生成订单号：ORD{yyyyMMddHHmmss}{随机8位}
        │  3. 从 Cart Service 获取购物车选中商品
        │  4. 从 Product Service 获取商品最新价格（防篡改）
        │  5. 计算金额：
        │     - 商品总价 = Σ(SKU单价 × 数量)
        │     - 运费 = 根据地区和重量计算
        │     - 优惠 = 优惠券抵扣 + 满减活动
        │     - 应付金额 = 商品总价 + 运费 - 优惠
        │  6. 创建 Order 记录（Status = Pending）
        │  7. 创建 OrderItem 记录（每个 SKU 一条）
        │  8. 保存订单快照地址（防止用户之后修改地址导致纠纷）
        │  9. 发布 OrderCreatedIntegrationEvent
        │
        ▼  [RabbitMQ] 消息投递到 order.created 队列
        │
[T2]    Inventory Service 消费 OrderCreated:
        │  1. 接收事件，开始本地事务
        │  2. 遍历 Order.Items：
        │     a. 查询库存（Redis 或 DB）
        │     b. 检查 AvailableQuantity >= OrderItem.Quantity
        │     c. 如果任一 SKU 库存不足：
        │        → 发布 InventoryFailedEvent
        │        → [进入补偿流程]
        │     d. 库存充足：ReservedQuantity += Quantity
        │  3. 记录 InventoryTransaction（类型=RESERVE）
        │  4. 提交事务
        │  5. 发布 InventoryReservedEvent（包含预留详情）
        │
        ├─ 成功路径 ─────────────────────────────────────────┐
        │                                                      │
[T3]    Cart Service 消费 InventoryReserved:                │
        │  1. 删除用户购物车中对应的商品                        │
        │  2. 发布 CartClearedEvent                            │
        │                                                      │
[T4]    Order Service 消费 CartCleared:                     │
        │  1. 更新订单 Status = AwaitingPayment                │
        │  2. 调用 Payment Service 创建支付单                  │
        │     → 返回支付参数（支付宝二维码URL/微信prepay_id）   │
        │  3. 将支付参数返回给前端                             │
        │                                                      │
[T5-T30] 用户扫码/确认支付（最长 30 分钟）                    │
        │                                                      │
[T31]   Payment Service 收到第三方支付成功回调:               │
        │  1. 验签                                            │
        │  2. 更新 Payment.Status = Paid                      │
        │  3. 发布 PaymentCompletedEvent                      │
        │                                                      │
[T32]   Order Service 消费 PaymentCompleted:                │
        │  1. 更新 Order.Status = Paid                        │
        │  2. 调用 Inventory Service 确认扣减                  │
        │     (ReservedQuantity → SoldQuantity)              │
        │  3. 发送通知：                                      │
        │     - 邮件：订单支付成功确认                         │
        │     - 站内信：支付成功                               │
        │     - 短信：（可选）支付成功通知                     │
        │  4. 返回成功响应给用户                               │
        │                                                      │
        ▼  ✅ 下单流程完成！                                   │
        │                                                      │
        └─ 失败/补偿路径 ────────────────────────────────────┘
        
        [任一步骤失败的补偿流程]
        
        Case A: 库存不足（T2 失败）
        ┌──────────────────────────────────────────┐
        │ Inventory Service 发布 InventoryFailed    │
        │         │                                │
        │         ▼                                │
        │ Order Service 接收失败事件:              │
        │  1. Order.Status = Cancelled             │
        │  2. 原因 = "库存不足"                     │
        │  3. 发布 OrderCancelledEvent             │
        │  4. 通知用户："抱歉，商品库存不足..."      │
        │  5. 购物车不清空（用户可稍后重试）        │
        └──────────────────────────────────────────┘

        Case B: 支付失败/超时（T5-T30 失败）
        ┌──────────────────────────────────────────┐
        │ Payment Service 发布 PaymentFailed/Expired│
        │         │                                │
        │         ▼                                │
        │ Order Service 接收失败事件:              │
        │  1. Order.Status = Expired               │
        │  2. 发布 OrderExpiredEvent               │
        │         │                                │
        │         ▼                                │
        │ Inventory Service 接收:                  │
        │  1. 释放预留库存（Reserved → Available） │
        │  2. 记录 InventoryTransaction(RELEASE)   │
        │         │                                │
        │         ▼                                │
        │ Notification Service:                    │
        │  通知用户："订单已超时关闭，库存已释放"    │
        └──────────────────────────────────────────┘
```

### 2. 补偿事务设计原则

1. **幂等性**：每个补偿操作可以安全地多次执行（如释放库存时先检查是否有预留）
2. **最终一致性**：不要求强一致，允许短暂的不一致窗口（通常 < 1秒）
3. **可观测性**：每个步骤都有日志记录和指标上报，便于排查问题
4. **超时机制**：每个 Saga 步骤都有超时限制（库存预留 10s / 支付创建 30s / 通知 5s）
5. **人工干预**：对于自动补偿失败的情况，提供管理员手动修复界面

---

## Docker Compose 一键部署

### 服务编排概览（15 个容器）

```yaml
# docker-compose.yml（精简版，完整版见 deploy/docker/ 目录）
version: '3.8'

services:

  # ===== 前端 =====
  frontend:
    build:
      context: ../CloudMall.Frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - api-gateway
    restart: unless-stopped

  # ===== API 网关 =====
  api-gateway:
    build:
      context: .
      dockerfile: src/ApiGateway/Dockerfile
    ports:
      - "8000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - IdentityService__Address=http://identity-service:8001
      - ProductService__Address=http://product-service:8002
      - OrderService__Address=http://order-service:8003
      - CartService__Address=http://cart-service:8004
      - PaymentService__Address=http://payment-service:8005
      - InventoryService__Address=http://inventory-service:8006
      - NotificationService__Address=http://notification-service:8007
    depends_on:
      - identity-service
      - product-service
      - order-service
      - cart-service
    restart: unless-stopped

  # ===== 8 个微服务 =====
  identity-service:
    build: src/IdentityService/
    ports:
      - "8001:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-identity;Port=5432;Database=cloudmall_identity;Username=postgres;Password=${DB_PASSWORD};
      - RedisSettings__ConnectionString=redis:6379
      - JwtSettings__SecretKeyPath=/run/secrets/jwt_private_key
    depends_on:
      - db-identity
      - redis
    volumes:
      - ./secrets:/run/secrets:ro
    restart: unless-stopped

  product-service:
    build: src/ProductService/
    ports:
      - "8002:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-products;Port=5432;Database=cloudmall_products;Username=postgres;Password=${DB_PASSWORD};
      - RedisSettings__ConnectionString=redis:6379
      - ElasticsearchSettings__Urls=http://elasticsearch:9200
      - MinioSettings__Endpoint=minio:9000
    depends_on:
      - db-products
      - redis
      - elasticsearch
      - minio
    restart: unless-stopped

  order-service:
    build: src/OrderService/
    ports:
      - "8003:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-orders;Port=5432;Database=cloudmall_orders;Username=postgres;Password=${DB_PASSWORD};
      - RedisSettings__ConnectionString=redis:6379
      - RabbitMQSettings__HostName=rabbitmq
      - MassTransit__RabbitMQ__Host=rabbitmq
      - CartService__Address=http://cart-service:8004
      - InventoryService__Address=http://inventory-service:8006
      - PaymentService__Address=http://payment-service:8005
    depends_on:
      - db-orders
      - redis
      - rabbitmq
    restart: unless-stopped

  cart-service:
    build: src/CartService/
    ports:
      - "8004:80"
    environment:
      - RedisSettings__ConnectionString=redis:6379
      - RabbitMQSettings__HostName=rabbitmq
    depends_on:
      - redis
      - rabbitmq
    restart: unless-stopped

  payment-service:
    build: src/PaymentService/
    ports:
      - "8005:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-payments;Port=5432;Database=cloudmall_payments;Username=postgres;Password=${DB_PASSWORD};
      - RedisSettings__ConnectionString=redis:6379
      - RabbitMQSettings__HostName=rabbitmq
    depends_on:
      - db-payments
      - redis
      - rabbitmq
    restart: unless-stopped

  inventory-service:
    build: src/InventoryService/
    ports:
      - "8006:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-inventory;Port=5432;Database=cloudmall_inventory;Username=postgres;Password=${DB_PASSWORD};
      - RedisSettings__ConnectionString=redis:6379
      - RabbitMQSettings__HostName=rabbitmq
    depends_on:
      - db-inventory
      - redis
      - rabbitmq
    restart: unless-stopped

  notification-service:
    build: src/NotificationService/
    ports:
      - "8007:80"
    environment:
      - ConnectionStrings__DefaultConnection=Host=db-notifications;Port=5432;Database=cloudmall_notifications;Username=postgres;Password=${DB_PASSWORD};
      - RabbitMQSettings__HostName=rabbitmq
      - EmailSettings__SmtpHost=${SMTP_HOST}
      - EmailSettings__SmtpPort=587
      - EmailSettings__SmtpUser=${SMTP_USER}
      - EmailSettings__SmtpPass=${SMTP_PASS}
    depends_on:
      - db-notifications
      - rabbitmq
    restart: unless-stopped

  # ===== 基础设施（数据库 × 4）=====
  db-identity:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_identity
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_identity:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 3s
      retries: 5

  db-products:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_products
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_products:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s

  db-orders:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_orders
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_orders:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  db-payments:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_payments
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_payments:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  db-inventory:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_inventory
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_inventory:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  db-notifications:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: cloudmall_notifications
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata_notifications:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]

  # ===== Redis =====
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s

  # ===== RabbitMQ =====
  rabbitmq:
    image: rabbitmq:3.12-management-alpine
    ports:
      - "5672:5672"    # AMQP 协议端口
      - "15672:15672"  # 管理界面 UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 10s

  # ===== Elasticsearch =====
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    ports:
      - "9200:9200"
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    healthcheck:
      test: ["CMD-SHELL", "curl -sf http://localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      retries: 10

  # ===== MinIO (S3 兼容对象存储) =====
  minio:
    image: minio/minio:latest
    ports:
      - "9000:9000"    # API
      - "9001:9001"    # Console
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: ${MINIO_PASSWORD}
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 10s

  # ===== 监控 =====
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=15d'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning:ro
    depends_on:
      - prometheus

volumes:
  pgdata_identity:
  pgdata_products:
  pgdata_orders:
  pgdata_payments:
  pgdata_inventory:
  pgdata_notifications:
  redis_data:
  rabbitmq_data:
  es_data:
  minio_data:
  prometheus_data:
  grafana_data:
```

---

## 运行步骤

### 前置条件

| 工具 | 最低版本 | 说明 |
|------|---------|------|
| Docker Desktop | 4.24+ | 包含 Docker Engine + Compose |
| .NET 8.0 SDK | 8.0.400+ | 编译所有微服务项目 |
| Git | 2.x | 克隆代码仓库 |
| 内存 | ≥ 16GB | 推荐（15个容器 + 数据库 + 缓存） |
| 磁盘空间 | ≥ 20GB | 镜像 + 数据卷 |
| CPU | ≥ 4 核 | 编译 + 运行 |

### 步骤 1：克隆代码仓库（1分钟）

```bash
git clone https://github.com/your-org/cloud-mall.git
cd cloud-mall
git submodule update --init --recursive  # 如果有子模块
```

### 步骤 2：配置环境变量（3分钟）

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件（填入你的配置）
# 必须修改的变量：
# - DB_PASSWORD: 数据库密码（至少12位）
# - RABBITMQ_PASSWORD: RabbitMQ 管理密码
# - MINIO_PASSWORD: MinIO 管理密码
# - GRAFANA_PASSWORD: Grafana 管理密码
# - JWT_SECRET_KEY: JWT 签名密钥（Base64 编码的 RSA 私钥）
# - SMTP_*: 邮件服务器配置（通知服务需要）
```

**.env 文件内容示例**：
```env
# ===== 数据库 =====
DB_PASSWORD=YourSecureDbPassword123!

# ===== RabbitMQ =====
RABBITMQ_PASSWORD=YourRabbitMQPassword

# ===== MinIO =====
MINIO_PASSWORD=YourMinIOAdminPassword

# ===== Grafana =====
GRAFANA_PASSWORD=YourGrafanaAdminPassword

# ===== JWT (RSA) =====
# 生成方式: openssl genrsa -out jwt_private.pem 2048
#          openssl rsa -in jwt_private.pem -pubout -outform PEM | base64 -w 0
JWT_PRIVATE_KEY_BASE64=<your-base64-encoded-rsa-private-key>

# ===== 邮件服务（通知服务需要）=====
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-specific-password

# ===== 支付（沙箱环境）=====
ALIPAY_APP_ID=sandbox_app_id
ALIPAY_PRIVATE_KEY=...
WECHAT_APP_ID=sandbox_app_id
WECHAT_MCH_ID=...
WECHAT_API_KEY=...

# ===== 短信（阿里云）=====
ACCESS_KEY_ID=your_access_key
ACCESS_KEY_SECRET=your_secret
SMS_SIGN_NAME=CloudMall
SMS_TEMPLATE_CODE=SMS_1234567890
```

### 步骤 3：构建所有镜像（10-20分钟，首次较慢）

```bash
# 构建所有服务镜像（并行构建）
docker compose build

# 或者单独构建某个服务（调试时更快）
docker compose build api-gateway identity-service product-service

# 查看构建进度
docker compose build --progress=plain  # 详细输出
```

### 步骤 4：启动基础设施服务（2分钟）

```bash
# 只启动数据库、Redis、RabbitMQ、ES、MinIO
docker compose up -d db-identity db-products db-orders db-payments db-inventory db-notifications redis rabbitmq elasticsearch minio

# 等待所有健康检查通过
docker compose ps  # 查看 status 是否都是 "healthy"

# 初始化数据库（在每个 db 容器中执行迁移）
docker compose exec db-identity psql -U postgres -d cloudmall_identity -f /docker-entrypoint-initdb.d/01-init.sql
# ... 对每个数据库执行类似的初始化命令
# 或者通过各个微服务自身的 dotnet ef database update 来初始化
```

### 步骤 5：启动所有微服务（2分钟）

```bash
# 启动全部 15 个服务
docker compose up -d

# 查看所有服务状态
docker compose ps

# 预期输出（所有服务应该是 Up/healthy）：
# NAME                          STATUS
# cloud-mall-frontend-1         running (healthy)
# cloud-mall-api-gateway-1      running (healthy)
# cloud-mall-identity-1         running (healthy)
# cloud-mall-product-1          running (healthy)
# cloud-mall-order-1            running (healthy)
# cloud-mall-cart-1             running (healthy)
# cloud-mall-payment-1          running (healthy)
# cloud-mall-inventory-1        running (healthy)
# cloud-mall-notification-1     running (healthy)
# cloud-mall-db-*-1             running (healthy) × 6
# cloud-mall-redis-1            running (healthy)
# cloud-mall-rabbitmq-1         running (healthy)
# cloud-mall-elasticsearch-1    running (healthy)
# cloud-mall-minio-1            running (healthy)
# cloud-mall-prometheus-1       running (healthy)
# cloud-mall-grafana-1          running (healthy)
```

### 步骤 6：初始化种子数据（2分钟）

```bash
# 通过 Identity Service API 创建管理员账户
curl -X POST http://localhost:8000/api/identity/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@cloudmall.com","password":"Admin@123456","userName":"admin"}'

# 通过 Product Service API 创建示例分类和商品（或使用种子数据脚本）
docker compose exec product-service dotnet run -- seed-data

# 通过 RabbitMQ Management UI 确认队列已创建
# 打开 http://localhost:15672 (admin / your RABBITMQ_PASSWORD)
# 查看 Queues 和 Exchanges 标签页
```

### 步骤 7：访问与验证

| 服务 | URL | 说明 |
|------|-----|------|
| **前端商城** | http://localhost:3000 | Vue.js 前端（Nginx 托管） |
| **Swagger API 文档** | http://localhost:8000/swagger | 网关聚合 Swagger（或直接访问各服务 /swagger） |
| **RabbitMQ 管理** | http://localhost:15672 | 消息队列管理界面 |
| **MinIO Console** | http://localhost:9001 | 对象存储管理界面 |
| **Grafana 监控** | http://localhost:3001 | 性能监控仪表盘（admin / GRAFANA_PASSWORD） |
| **Prometheus** | http://localhost:9090 | 指标查询界面 |
| **Elasticsearch** | http://localhost:9200 | 搜索引擎（直接 JSON API） |

**健康检查**：
```bash
# 网关健康检查
curl http://localhost:8000/api/health

# 各服务健康检查
curl http://localhost:8001/api/health  # Identity
curl http://localhost:8002/api/health  # Product
curl http://localhost:8003/api/health  # Order
# ...

# 端到端测试：注册 → 登录 → 浏览商品 → 加入购物车 → 下单
```

### 常用运维命令

```bash
# 查看所有服务日志（跟踪模式）
docker compose logs -f

# 查看特定服务日志
docker compose logs -f order-service

# 重启某个服务
docker compose restart order-service

# 扩展某个服务（水平扩展，需配合负载均衡器）
docker compose up -d --scale order-service=3

# 停止所有服务
docker compose down

# 停止并删除数据卷（⚠️ 会丢失所有数据！）
docker compose down -v

# 查看资源使用情况
docker stats

# 进入某个服务容器内部调试
docker compose exec order-service sh
```

---

## 学习要点（30+ 知识点）

### 微服务架构设计（DDD）
1. **服务拆分原则**：按业务能力拆分（而非技术分层）； bounded context 边界；单一职责 + 高内聚低耦合
2. **领域驱动设计（DDD）实践**：Aggregate Root（聚合根）、Entity vs Value Object、Domain Event（领域事件）、Repository Pattern
3. **API Gateway 模式**：YARP 反向代理；BFF（Backend for Frontend）聚合；统一认证/限流/日志
4. **服务间通信**：同步 REST（HTTP Client + Polly 弹性）vs 异步消息（RabbitMQ/MassTransit）；何时用哪种？

### 分布式事务与 Saga
5. **Saga 模式**：Choreography（ choreography 编排，事件驱动）vs Orchestration（ orchestration 编排，中央协调器）；本项目采用 Orchestration（MassTransit StateMachine）
6. **补偿事务（Compensating Transaction）**：正向操作的逆向操作；保证最终一致性；补偿操作的幂等性
7. **幂等性设计**：Idempotency Key；去重表；Redis SETNX；数据库唯一约束
8. **分布式锁**：Redis RedLock 算法；库存扣减场景；锁的超时和续约

### 消息队列（RabbitMQ）
9. **MassTransit 框架**：Publish/Consume/Send/Respond；Consumer/Producer/Saga State Machine；消息序列化（JSON/protobuf）
10. **RabbitMQ Exchange 类型**：Direct（路由键精确匹配）、Topic（通配符匹配）、Fanout（广播）、Headers（消息头匹配）
11. **消息可靠性**：Publisher Confirm（发布确认）、Consumer Ack（手动确认）、DLQ（死信队列）、消息持久化 + 镜像队列
12. **消息顺序性**：单队列单消费者保证顺序；分区有序（按 OrderId 分区）

### 数据管理与一致性
13. **每服务独立数据库（Database per Service）**：PostgreSQL × 6；跨服务查询的挑战与解决方案（API 聚合 / CDC / 读副本）
14. **分布式数据一致性**：CAP 理论权衡（CP vs AP）；BASE 理论（基本可用、软状态、最终一致性）；一致性级别选择
15. **缓存策略（多级缓存）**：本地缓存（MemoryCache）→ 分布式缓存（Redis）→ 数据库；Cache-Aside / Read-Through / Write-Through / Write-Behind
16. **Redis 在微服务中的应用**：购物车存储（Hash）、会话存储（String JSON）、分布式锁（SET NX EX）、限流计数器（INCR + TTL）、Pub/Sub（轻量消息）

### 安全体系
17. **JWT RSA 非对称认证**：私钥签署（Identity Service）+ 公钥验证（Gateway + 各服务）；密钥轮换策略
18. **OAuth2 / OpenID Connect**：第三方登录流程（Authorization Code Flow）；IdentityServer4 / Duende 配置
19. **RBAC 权限模型**：角色-权限矩阵；自定义 AuthorizationHandler；Resource-based Authorization（Owner 可操作自己的资源）
20. **微服务安全最佳实践**：Service-to-Service 认证（Client Credentials Grant）；内部 API 不对外暴露；Network Policy（Kubernetes）或 Docker Network 隔离

### 弹性与容错（Polly）
21. **弹性策略组合**：Retry（重试：指数退避）→ Circuit Breaker（熔断：半开/开启/关闭）→ Bulkhead Isolation（舱壁隔离：线程池/信号量隔离）→ Timeout（超时）→ Fallback（降级）
22. **HttpClientFactory + Polly**：Typed Client；策略注册；动态策略更新
23. **服务降级策略**：商品搜索降级为数据库 LIKE（ES 不可用时）；推荐服务降级为热门榜单；购物车降级为只读模式

### 可观测性（Observability）
24. **三大支柱**：Logs（结构化日志 Serilog）+ Metrics（Prometheus Counter/Gauge/Histogram）+ Traces（OpenTelemetry + Jaeger Distributed Tracing）
25. **分布式链路追踪**：TraceId/SpanId/ParentSpanId 传播（HTTP Header + RabbitMQ Header）；Span Kind（Client/Server/Producer/Consumer）；性能瓶颈定位
26. **监控仪表盘（Grafana）**：Dashboard 模板（服务 QPS / 延迟 P50-P99 / 错误率 / 饱和度 / 资源使用率）；Alert Rule（延迟 > 1s 告警 / 错误率 > 1% 告警 / CPU > 80% 告警）

### 容器化与 DevOps
27. **Docker 多阶段构建**：SDK 构建 → Runtime 运行；镜像大小优化（alpine 基础镜像 / 删除不必要的文件）；.dockerignore
28. **Docker Compose 编排**：服务依赖（depends_on + healthcheck）；网络隔离（bridge 网络）；数据持久化（named volumes）；环境变量注入（.env 文件 / Docker Secrets）；健康检查
29. **CI/CD 流程**（GitHub Actions / Azure DevOps）：Lint → Unit Test → Build Docker Image → Push to Registry → Deploy to Staging → Integration Test → Promote to Production（Blue-Green / Canary）
30. **基础设施即代码（IaC）**：Docker Compose 作为开发/测试环境的 IaC；生产环境升级到 Kubernetes（Deployment / Service / Ingress / ConfigMap / Secret / HPA）

### 业务领域知识
31. **电商领域模型**：SPU/SKU 设计；商品分类树；购物车状态机；订单状态机（10 种状态）；支付流程与回调；库存并发控制
32. **推荐系统基础**：基于内容的推荐；协同过滤；热门排行；A/B 测试框架

---

## 项目统计

| 指标 | 数值 |
|------|------|
| **总代码量** | ~15,000 行（C# + 前端 + 配置 + 测试） |
| **微服务数量** | 8 个独立服务 + 1 个 API Gateway |
| **API 端点总数** | 60+ 个 RESTful API |
| **数据库实例** | 6 个 PostgreSQL 数据库 |
| **数据库表总数** | 20+ 张表 |
| **RabbitMQ 队列** | 10+ 个业务队列 |
| **Docker 容器** | 15 个容器（8 服务 + 6 DB + Redis） |
| **前置知识要求** | ASP.NET Core MVC → EF Core → RESTful API → Docker 基础 |
| **建议学习路径** | Hello World → Todo App → Blog System → **CloudMall（本项目）** |
| **预计学习耗时** | 30-40 小时（含阅读源码、动手实验、调试排错） |

---

## 前置知识要求

在开始本项目之前，请确保你已经掌握以下技能：

### 必须掌握（Prerequisites）
- [ ] **C# 12 语言特性**：record 类型、pattern matching、primary constructor、collection expressions、nullable reference types
- [ ] **ASP.NET Core MVC 基础**：Controller/View/Model、Razor 语法、Middleware 管道、DI 容器
- [ ] **Entity Framework Core**：Code First Migration、LINQ 查询、关系配置（1:N、N:M、自引用）、Change Tracking
- [ ] **RESTful API 设计**：HTTP 方法语义、状态码、资源命名规范、分页/排序/过滤
- [ ] **异步编程**：async/await、Task、CancellationToken、ConfigureAwait
- [ ] **Git 版本控制**：branch/merge/pull request/rebase

### 推荐掌握（Recommended）
- [ ] **Docker 基础**：Dockerfile 编写、docker-compose.yml、镜像/容器/卷/网络概念
- [ ] **Redis 基础**：String/Hash/List/Set/ZSet 数据结构、TTL、发布订阅
- [ ] **SQL 基础**：PostgreSQL 或 MySQL 熟练使用、索引优化、事务隔离级别
- [ ] **Linux 基础**：常用命令（ls/grep/tail/chmod/systemctl）、Shell 脚本入门

### 可以边学边用（Learn as You Go）
- [ ] **消息队列概念**：Producer/Consumer/Exchange/Queue/Binding
- [ ] **微服务理论**：CAP/BASE、Saga、CQRS、Event Sourcing、API Gateway
- [ ] **监控可观测性**：Metrics/Logging/Tracing、PromQL、Grafana 配置
- [ ] **Kubernetes 基础**（本项目用 Docker Compose，K8s 为进阶可选）：Pod/Service/Deployment/Ingress

---

## 关联教程索引

| 知识域 | 教程位置 | 本项目应用位置 |
|--------|---------|---------------|
| 微服务架构概述 | [[高手篇/01-微服务架构设计]] | 整体架构设计 |
| DDD 领域驱动设计 | [[高手篇/02-DDD实战]] | 各服务领域模型 |
| API 网关 (YARP) | [[高手篇/04-API网关]] | ApiGateway 项目 |
| 分布式事务 (Saga) | [[高手篇/05-Saga分布式事务]] | Order Service Saga 编排器 |
| 消息队列 (RabbitMQ) | [[高手篇/06-消息队列实战]] | MassTransit 集成 |
| Docker 容器化 | [[高手篇/03-Docker容器化部署]] | Docker Compose 编排 |
| 可观测性 | [[高手篇/08-可观测性与监控]] | Prometheus + Grafana + Jaeger |
| CI/CD | [[高手篇/09-CI_CD流水线]] | GitHub Actions 配置 |
| JWT RSA 认证 | [[进阶篇/02-JWT认证详解]] | Identity Service |
| EF Core 高级 | [[进阶篇/04-EF-Core高级特性]] | 各服务 Repository |
| Polly 弹性策略 | [[进阶篇/10-弹性与容错]] | HttpClientFactory 配置 |
| Vue.js 前端集成 | [[进阶篇/09-前后端分离实战]] | CloudMall.Frontend |

---

## 许可证

本项目仅用于学习和教育目的。代码基于 MIT 许可证开源。

---

**最后更新时间**：2026-04-18
**维护者**：ASP.NET Core 知识库团队
**复杂度警告**：⚠️ 这是知识库中最复杂的项目，建议按照 Hello World → Todo App → Blog System → **CloudMall** 的顺序逐步学习。不要试图一次性理解所有概念，建议每次聚焦于 1-2 个微服务深入理解。
