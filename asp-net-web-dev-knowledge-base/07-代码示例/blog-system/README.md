# 博客系统（微服务架构预备版）

> **对应教程**：[[进阶篇/06-实战项目-博客系统]]
> **难度等级**：⭐⭐⭐⭐ 高级 | **预计耗时**：15-20小时
> **适用人群**：已完成 Todo 应用，希望掌握 RESTful API、JWT 认证、富文本编辑和全文搜索的开发者

---

## 项目概述

这是一个**功能完整的现代化博客系统**，采用前后端分离架构（ASP.NET Core Web API + Vue.js 3 / Blazor），集成了 JWT Bearer Token 认证、Markdig Markdown 解析引擎、嵌套评论系统、Elasticsearch 全文搜索、Redis 缓存加速、MinIO 对象存储等企业级技术栈。

本项目不仅是一个功能丰富的博客平台，更是**从单体 MVC 应用向微服务架构过渡的桥梁项目**。它展示了如何设计 RESTful API、如何实现安全的认证授权机制、如何处理复杂的数据关系（多对多、树形结构）、如何优化查询性能（缓存 + 索引 + 全文搜索），以及如何构建可扩展的后端服务。

**核心价值**：
- 掌握 RESTful API 设计规范与最佳实践
- 理解 JWT OAuth2.0 认证体系的完整流程
- 学会处理富文本内容（Markdown → HTML 转换、XSS 防护）
- 掌握嵌套数据结构的存储与查询算法
- 建立生产级系统的安全意识和性能优化能力

---

## 技术栈

### 后端技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **ASP.NET Core Web API** | 8.0 | RESTful API 框架（纯 API，无视图渲染） |
| **Entity Framework Core** | 8.0 | ORM 框架，复杂数据关系映射 |
| **SQL Server** | 2019+ / Azure SQL | 主数据库（用户、文章、评论等） |
| **Markdig** | 0.38.x | Markdown → HTML 转换引擎 |
| **JWT Bearer (System.IdentityModel.Tokens.Jwt)** | 7.x | JSON Web Token 认证 |
| **StackExchange.Redis** | 2.7.x | 分布式缓存（会话、热点数据） |
| **MinIO (Minio)** | 4.0.x | S3 兼容对象存储（图片上传） |
| **Serilog** | 3.1.x | 结构化日志框架 |
| **FluentValidation** | 11.8.x | 高级请求验证 |
| **AutoMapper** | 12.0.x | 对象映射（Entity ↔ DTO） |
| **Swashbuckle.AspNetCore (Swagger)** | 6.5.x | API 文档自动生成 |
| **xUnit + Moq** | 最新版 | 单元测试与集成测试 |

### 前端技术栈（二选一）

#### 方案 A：Vue.js 3（推荐新手）
| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **Vue.js 3** | 3.4+ | 渐进式 JavaScript 框架（Composition API） |
| **Vite** | 5.x | 构建工具（极速热更新） |
| **Vue Router 4** | 4.x | 客户端路由 |
| **Pinia** | 2.x | 状态管理（替代 Vuex） |
| **Axios** | 1.x | HTTP 客户端（API 请求） |
| **Element Plus** | 2.5.x | UI 组件库（类似 Ant Design） |
| **Markdown 编辑器** | md-editor-v3 / ByteMD | 所见即所得编辑器 |
| **Highlight.js** | 11.x | 代码语法高亮 |

#### 方案 B：Blazor WebAssembly（推荐 .NET 开发者）
| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **Blazor WebAssembly** | 8.0 | 基于 WebAssembly 的 .NET 前端框架 |
| **BlazorBootstrap** | 最新版 | UI 组件库 |

### 基础设施技术栈

| 技术 | 版本 | 用途说明 |
|------|------|----------|
| **Docker & Docker Compose** | 最新版 | 容器化部署与编排 |
| **Nginx** | 1.24+ | 反向代理 + 静态资源服务 + SSL 终结 |
| **Let's Encrypt (Certbot)** | 最新版 | 免费 HTTPS 证书自动续期 |

---

## 功能清单

### 核心功能模块（7大模块，30+ RESTful API）

---

#### 1️⃣ 用户模块（JWT 注册登录 + Refresh Token + 密码修改）

##### 1.1 用户注册（POST /api/auth/register）
- **请求字段**：username（3-20字符）、email（格式验证+唯一性）、password（8位+大小写+数字）、confirmPassword
- **业务逻辑**：用户名/邮箱唯一性检查、密码强度验证、bcrypt 哈希存储（工作因子12）、默认角色分配、欢迎邮件发送（可选）

##### 1.2 用户登录（POST /api/auth/login）
- **支持双字段登录**：用户名或邮箱均可
- **安全措施**：bcrypt 凭据验证、账户状态检查、登录失败次数限制（5次失败锁定15分钟）
- **Token 生成**：JWT Access Token（15分钟有效期）+ Refresh Token（7天，HttpOnly Cookie）
- **响应示例**：
```json
{
  "success": true,
  "message": "登录成功",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "tokenType": "Bearer",
    "expiresIn": 900,
    "user": { "id": "usr_abc123", "username": "johndoe", "roles": ["User"] }
  }
}
```

##### 1.3 刷新 Token（POST /api/auth/refresh-token）
- 从 HttpOnly Cookie 读取 Refresh Token，验证有效性后生成新 Token 对（轮换机制）

##### 1.4 退出登录（POST /api/auth/logout）
- 将 Refresh Token 加入 Redis 黑名单（TTL = 剩余有效期），清除客户端 Cookie

##### 1.5 修改密码（PUT /api/auth/password）
- 验证当前密码、新密码强度检查、撤销所有现有 Refresh Token

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/auth/register` | 用户注册 | 公开 |
| POST | `/api/auth/login` | 用户登录 | 公开 |
| POST | `/api/auth/refresh-token` | 刷新 Token | 公开（需 Cookie） |
| POST | `/api/auth/logout` | 退出登录 | 已认证 |
| PUT | `/api/auth/password` | 修改密码 | 已认证 |
| GET | `/api/users/me` | 获取当前用户 | 已认证 |
| PUT | `/api/users/me` | 更新用户资料 | 已认证 |

---

#### 2️⃣ 文章模块（CRUD + 富文本 Markdown + Slug + 状态机 + 草稿发布 + 软删除）

##### 2.1 创建文章（POST /api/articles）
- **权限**：已认证用户（Author 角色以上）
- **核心处理流程**：
  - 标题/内容验证 → 自动生成摘要（前200字符提取）→ 自动生成 Slug（URL友好标识符，全局唯一）
  - **Markdown 转 HTML**：使用 Markdig 将 content 转换为 HTML 存储（用于显示），保留原始 Markdown（用于编辑）
  - **XSS 防护**：HTML 输出使用 sanitize-html 过滤危险标签（script、iframe、onclick 等）
  - 状态初始化（Draft/Published）、阅读时间估算（中文300字/分钟）、记录作者信息

##### 2.2 获取文章列表（GET /api/articles）
- **权限过滤**：未认证用户只能看到 Published 状态；已认证用户可看到自己的 Draft
- **查询参数**：page/pageSize/categoryId/tagId/status/authorId/keyword/sortBy/sortOrder
- **缓存策略**：热门文章列表缓存到 Redis（TTL 5分钟）；Select 投影减少传输量

##### 2.3 获取文章详情（GET /api/articles/{id} 或 /slug/{slug})
- 支持通过 ID 或 Slug 访问（SEO 友好）
- **递增阅读量**：使用 Redis INCR 原子操作，每5分钟批量同步到数据库
- **附加数据**：目录生成（TOC，从标题提取）、上一篇/下一篇、推荐文章（标签相似度 Jaccard 算法 Top 5）、作者其他文章

##### 2.4 更新文章（PUT /api/articles/{id})
- 乐观并发控制（RowVersion/timestamp）、Slug 变更处理、状态转换验证、版本历史快照保存（ArticleVersion 表，支持回滚）

##### 2.5 删除文章（DELETE /api/articles/{id})
- 软删除（status=Deleted）或硬删除（仅管理员）；清除相关缓存；记录操作日志

##### 2.6 文章状态机

```
                    ┌─────────────┐
                    │   Draft     │ ← 创建初始状态
                    └──────┬──────┘
                           │ 发布(Publish)
                           ▼
                    ┌─────────────┐
              ┌───▶│  Published  │◀──┐
              │    └──┬─────┬────┘   │
              │  归档  │      │ 下架  │
              │       ▼      ▼       │
              │  ┌──────────┐ ┌────┴────┐
              │  │ Archived │ │ Hidden  │
              │  └──────────┘ └────┬────┘
              │                      │ 重新发布
              │                ┌────▼────┐
              └────────────────│ Deleted │
                               └──────────┘
```

**允许的状态转换**：Draft→Published/Deleted, Published→Archived/Hidden/Deleted, Archived→Published, Hidden→Published, Deleted→Draft(仅管理员)

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/articles` | 创建文章 | 已认证（Author+） |
| GET | `/api/articles` | 获取文章列表 | 公开 |
| GET | `/api/articles/{id}` | 获取文章详情（ByID） | 公开 |
| GET | `/api/articles/slug/{slug}` | 获取文章详情（BySlug） | 公开 |
| PUT | `/api/articles/{id}` | 更新文章 | 作者/管理员 |
| DELETE | `/api/articles/{id}` | 删除文章（软删除） | 作者/管理员 |
| PUT | `/api/articles/{id}/status` | 更改文章状态 | 作者/管理员 |
| GET | `/api/articles/{id}/versions` | 获取版本历史 | 作者/管理员 |
| POST | `/api/articles/{id}/restore` | 恢复已删除文章 | 管理员 |

---

#### 3️⃣ 评论模块（嵌套评论最多 5 层 + 树形构建算法 + 审核机制）

##### 评论数据模型
```csharp
public class Comment
{
    public int Id { get; set; }                              // 主键
    public string Content { get; set; } = string.Empty;      // 评论内容
    public int? ParentId { get; set; }                       // 父评论 ID（null=根评论）
    public int ArticleId { get; set; }                       // 所属文章 ID
    public string UserId { get; set; } = string.Empty;       // 评论者用户 ID
    public CommentStatus Status { get; set; } = Pending;     // Pending/Approved/Rejected/Deleted
    public int LikeCount { get; set; }                        // 点赞数
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    
    // 导航属性（自引用）
    public Comment? Parent { get; set; }
    public ICollection<Comment> Children { get; set; } = new List<Comment>();
}
```

##### 发布评论（POST /api/articles/{articleId}/comments）
- 内容验证（1-1000字符）、嵌套深度检查（最多5层递归查询ParentId链）
- 频率限制（同用户60秒内最多3条，Redis计数器）
- IP和UA记录、自动审核（可信用户自动Approved）

##### 获取评论树（GET /api/articles/{articleId}/comments）
- **两阶段查询**：先查根评论分页，再查所有后代评论（避免N+1问题）
- **树形构建算法**（内存中构建，O(n)时间复杂度）：
```csharp
public List<CommentTreeNode> BuildTree(List<Comment> flatComments)
{
    var nodeDict = flatComments.ToDictionary(c => c.Id);
    var roots = new List<CommentTreeNode>();
    
    foreach (var comment in flatComments)
    {
        var node = MapToDto(comment);
        if (comment.ParentId == null || !nodeDict.ContainsKey(comment.ParentId.Value))
            roots.Add(node);  // 根节点
        else
            nodeDict[comment.ParentId.Value].Children.Add(node);  // 添加到父节点
    }
    
    return roots;
}
```
- 深层嵌套（>3层）默认折叠；热门文章评论树缓存到Redis（TTL 2分钟）

##### 其他功能：删除评论（软删除）、点赞/取消点赞（toggle，Redis Set）、批量审核（管理员）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/articles/{articleId}/comments` | 发布评论 | 已认证 |
| GET | `/api/articles/{articleId}/comments` | 获取评论树 | 公开 |
| DELETE | `/api/comments/{commentId}` | 删除评论 | 本人/管理员 |
| POST | `/api/comments/{commentId}/like` | 点赞/取消点赞 | 已认证 |
| PUT | `/api/comments/{commentId}/status` | 审核评论 | 管理员 |

---

#### 4️⃣ 标签与分类（多对多关系 + 标签云 + 分类合并）

##### 分类（Categories）
- CRUD 操作、两级分类结构（父子分类）、Slug自动生成、文章计数、手动排序

##### 标签（Tags）
- CRUD 操作、多对多关系（ArticleTag中间表，EF Core原生支持跳表）
- **标签云生成算法**：按使用频率线性映射到字体大小（1.0em~3.0em）和颜色渐变（冷色→暖色）
- 热门Top 20缓存到Redis（每小时刷新）、输入自动补全（前缀匹配）
- 标签合并功能（迁移所有关联关系）

##### EF Core 多对多配置
```csharp
modelBuilder.Entity<Article>()
    .HasMany(a => a.Tags)
    .WithMany(t => t.Articles)
    .UsingEntity<Dictionary<string, object>>(
        "ArticleTag",
        j => j.HasOne<Tag>().WithMany().HasForeignKey("TagId"),
        j => j.HasOne<Article>().WithMany().HasForeignKey("ArticleId"));
```

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/categories` | 获取分类列表（树形） | 公开 |
| POST | `/api/categories` | 创建分类 | 管理员 |
| GET | `/api/tags` | 获取标签列表 | 公开 |
| GET | `/api/tags/cloud` | 获取标签云 | 公开 |
| GET | `/api/tags/suggest` | 标签自动补全 | 公开 |
| PUT | `/api/tags/{id}/merge` | 合并标签 | 管理员 |

---

#### 5️⃣ 图片上传（三重验证 + 缩略图 + WebP 转换 + 批量上传）

##### 三重验证机制
1. **客户端验证**（JavaScript）：文件类型accept="image/*"、大小<10MB
2. **服务端验证**（C#）：文件存在性、大小限制、扩展名白名单（jpg/png/gif/webp）、Magic Number检测防伪造扩展名
3. **病毒扫描**（可选生产环境）：集成ClamAV或云服务商恶意文件检测API

##### 图片处理流水线
1. 生成唯一文件名（GUID）
2. 根据UploadType调整尺寸（封面1200x630 OG标准/内容1024px宽/头像256x256正方形裁剪）
3. 生成缩略图（150x150）
4. 编码为WebP格式（比JPEG小25-35%，比PNG小80%）
5. 上传到MinIO（S3兼容对象存储）
6. 返回CDN访问URL

##### 批量上传：一次最多9张，原子性（任一失败全部回滚）

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| POST | `/api/upload/images` | 单张图片上传 | 已认证 |
| POST | `/api/upload/images/batch` | 批量上传（最多9张） | 已认证 |
| DELETE | `/api/upload/images/{filename}` | 删除图片 | 上传者/管理员 |
| GET | `/api/upload/presigned-url` | 预签名上传URL（直传） | 已认证 |

---

#### 6️⃣ 搜索功能（多字段组合 + 高亮 + 热门搜索 + 搜索历史）

##### 全文搜索（GET /api/search?q=keyword）
- **方案A**：SQL Server Full-Text Search（简单场景，<10万篇文章）— CONTAINSTABLE
- **方案B**（推荐）：Elasticsearch（大规模场景）— 多字段匹配（标题权重3x、摘要2x、内容1x）、模糊匹配（Fuzziness.Auto容忍拼写错误）、高亮片段（mark标签包裹）

##### 关键词高亮
```css
.search-highlight {
    background-color: #fff59d;
    padding: 0 2px;
    border-radius: 2px;
}
```

##### 热门搜索：Redis Sorted Set（Key: `search:trending`, Score: 搜索次数），Top 20 + 趋势指示（上升/下降/持平）

##### 搜索历史：Redis List（Key: `search:history:{userId}`, 最多50条），支持查看/删除单条/清空

##### 搜索建议：输入≥2字符触发（300ms防抖），Elasticsearch Completion Suggester 或 SQL LIKE前缀匹配

**API 端点汇总**：

| 方法 | 路径 | 描述 | 权限 |
|------|------|------|------|
| GET | `/api/search` | 全文搜索 | 公开 |
| GET | `/api/search/hot` | 热门搜索 | 公开 |
| GET | `/api/search/history` | 搜索历史 | 已认证 |
| GET | `/api/search/suggest` | 搜索建议 | 公开 |

---

#### 7️⃣ 部署上线（IIS / Docker / Azure 三种方案）

详见"运行步骤"部分。

---

## 5 实体 ER 图说明

### 数据库实体关系概览

```
┌──────────────────┐       ┌──────────────────────────┐
│     User (用户)    │       │      Article (文章)        │
│──────────────────│       │──────────────────────────│
│ PK Id (GUID)      │       │ PK Id (int, 自增)          │
│ UserName          │       │ FK AuthorId → User.Id      │
│ Email             │       │ FK CategoryId → Category.Id│
│ PasswordHash      │       │ Slug (unique)              │
│ NickName/Avatar   │       │ Title/Content(Markdown)    │
│ Role/Status       │       │ ContentHtml/Status         │
└────────┬─────────┘       │ ViewCount/LikeCount        │
         │                 │ ReadingTime/RowVersion      │
         │ 1               └──────┬───────────┬─────────┘
         │                        │           │
         │ N                      │ N         │ N
         │                        ▼           ▼
         │              ┌──────────┐  ┌─────────┐
         │              │ Tag (标签) │  │ArticleVer│
         │              │──────────│  │(版本历史) │
         ▼              │ Name/Slug │  └─────────┘
┌──────────────────┐    │ UsageCount│
│  Comment (评论)    │    └────┬─────┘
│──────────────────┘         │ N:N
│ PK Id (int)       │   (中间表 ArticleTag)
│ FK ArticleId      │        ▼
│ FK UserId         User  ┌────────────────┐
│ FK ParentId(self) │◀──┐ │  Category (分类) │
│ Content/Status    │   │ │────────────────│
│ LikeCount/CreatedAt│   │ │ Id/Name/Slug    │
└──────────────────┘   │ │ FK ParentId(self)│ ◄── 自引用
                       │ │ DisplayOrder    │
                       │ └────────────────┘
```

### 实体详细定义

#### 1. User（用户表，28个字段）
关键字段：Id(GUID)、UserName(Unique)、Email(Unique)、PasswordHash(bcrypt)、NickName、AvatarUrl、Bio、Role(User/Author/Admin/Moderator)、Status(正常/禁用/锁定)、LastLoginAt、LoginFailCount、LockoutEnd、SecurityStamp、TwoFactorEnabled、CreatedAt/UpdatedAt

#### 2. Article（文章表，25个字段）
关键字段：Id(int自增)、AuthorId(FK)、CategoryId(FK)、Slug(Unique)、Title、Content(Markdown)、ContentHtml(HTML)、Summary、CoverImage、Status(Draft/Published/Archived/Hidden/Deleted)、IsDeleted、ViewCount/LikeCount/CommentCount/FavoriteCount、ReadingTime、IsTop、AllowComment、PublishedAt、RowVersion(timestamp)、CreatedAt/UpdatedAt

#### 3. Category（分类表，11个字段）
关键字段：Id、Name、Slug(Unique)、Description、ParentId(FK自引用, Null=顶级)、DisplayOrder、CoverImage、ArticleCount(冗余)、CreatedAt/UpdatedAt

#### 4. Tag（标签表，7个字段）+ 中间表 ArticleTag
关键字段：Id、Name、Slug(Unique)、Description、UsageCount(冗余)、CreatedAt
中间表：(ArticlesId, TagsId) 复合主键

#### 5. Comment（评论表，13个字段，含自引用外键）
关键字段：Id、Content、ArticleId(FK)、UserId(FK)、ParentId(FK自引用, Null=根评论)、Status(Pending/Approved/Rejected/Deleted)、LikeCount、IpAddress、UserAgent、CreatedAt/UpdatedAt

---

## 统一响应格式

整个 API 采用统一的 `ApiResponse<T>` 包装格式：

```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }           // 是否成功
    public int StatusCode { get; set; }         // HTTP 状态码
    public string Message { get; set; }         // 消息描述
    public T? Data { get; set; }                // 数据载荷
    public IEnumerable<ApiError>? Errors { get; set; }  // 错误详情
    public PaginationInfo? Pagination { get; set; }      // 分页信息（列表接口）
    public string TraceId { get; set; }         // 请求追踪ID
    public string Timestamp { get; set; }       // 服务器时间戳（ISO 8601）
}
```

**成功响应示例**（列表含分页）：
```json
{
  "success": true,
  "statusCode": 200,
  "message": "查询成功",
  "data": [{ "id": 42, "title": "..." }],
  "pagination": { "currentPage": 1, "pageSize": 10, "totalCount": 156, "totalPages": 16 },
  "traceId": "00-xyz789",
  "timestamp": "2026-04-17T10:30:00Z"
}
```

**错误响应示例**（400 验证错误）：
```json
{
  "success": false,
  "statusCode": 400,
  "message": "请求参数验证失败",
  "errors": [
    { "field": "Title", "code": "REQUIRED", "message": "标题不能为空" },
    { "field": "Email", "code": "FORMAT_INVALID", "message": "邮箱格式不正确" }
  ]
}
```

---

## 安全体系（10项安全措施）

| # | 安全措施 | 实现方式 | 重要程度 |
|---|---------|---------|---------|
| 1 | **JWT Bearer Token 认证** | JWT + Refresh Token 轮换 + HttpOnly Cookie | ★★★★★ |
| 2 | **密码安全存储** | bcrypt 哈希（工作因子 12） | ★★★★★ |
| 3 | **HTTPS 强制加密** | HSTS + TLS 1.2+ + Let's Encrypt | ★★★★★ |
| 4 | **CORS 跨域策略** | 白名单域名 + 预检请求缓存 | ★★★★☆ |
| 5 | **XSS 防护** | Markdown HTML 白名单过滤 + CSP 头 | ★★★★★ |
| 6 | **CSRF 防护** | SameSite Cookie + Anti-CSRF Token | ★★★★☆ |
| 7 | **SQL 注入防护** | EF Core 参数化查询 | ★★★★★ |
| 8 | **速率限制** | 固定窗口算法（Redis 计数器）：登录5次/分、评论3次/分、上传10次/分 | ★★★★☆ |
| 9 | **输入验证** | FluentValidation + Data Annotations 双重验证 | ★★★★★ |
| 10 | **安全响应头** | X-Frame-Options/SameSite/X-XSS-Protection/Content-Security-Policy/Strict-Transport-Security | ★★★★☆ |

### JWT 配置要点

```csharp
// Access Token 结构
var tokenDescriptor = new SecurityTokenDescriptor
{
    Subject = new ClaimsIdentity(new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.Id),
        new Claim(JwtRegisteredClaimNames.UniqueName, user.UserName),
        new Claim(ClaimTypes.Role, user.Role),
        new Claim("jti", Guid.NewGuid().ToString())  // 唯一ID用于吊销
    }),
    Expires = DateTime.UtcNow.AddMinutes(15),  // 短有效期降低泄露风险
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature),
    Issuer = "https://api.myblog.com",
    Audience = "https://myblog.com"
};
```

---

## 目录结构

```
blog-system/
├── BlogSystem.sln
│
├── BlogSystem.Api/                          # 🎯 后端 Web API 项目
│   ├── Program.cs                           # 应用入口（DI + 中间件管道）
│   ├── appsettings.json                     # 配置文件
│   ├── Controllers/                         # 9个API控制器
│   │   ├── AuthController.cs               # 认证（注册/登录/刷新/登出/改密）
│   │   ├── ArticlesController.cs           # 文章 CRUD
│   │   ├── CategoriesController.cs         # 分类管理
│   │   ├── TagsController.cs               # 标签管理
│   │   ├── CommentsController.cs           # 评论管理
│   │   ├── SearchController.cs             # 搜索功能
│   │   ├── UploadController.cs             # 文件上传
│   │   ├── UsersController.cs              # 用户管理
│   │   └── Admin/AdminDashboardController.cs # 管理后台
│   │
│   ├── Models/
│   │   ├── Entities/                        # 7个数据库实体
│   │   │   ├── User.cs / Article.cs / ArticleVersion.cs
│   │   │   ├── Category.cs / Tag.cs / Comment.cs
│   │   │   └── RefreshToken.cs
│   │   ├── Enums/                           # ArticleStatus/CommentStatus/UserRole/UploadType
│   │   ├── DTOs/                            # 8类DTO（Auth/User/Article/Category/Tag/Comment/Search/Upload/Pagination）
│   │   ├── Requests/                        # 8个请求模型
│   │   └── Common/                          # ApiResponse/ApiError/PagedResult/PaginationInfo
│   │
│   ├── Services/                            # 业务逻辑层（接口+实现）
│   │   ├── Interfaces/ (9个服务接口)
│   │   ├── Implementations/ (9个服务实现)
│   │   └── Profiles/ (AutoMapper配置)
│   │
│   ├── Repositories/                        # 数据访问层
│   │   ├── Interfaces/ (泛型IRepository + 6个专用接口)
│   │   ├── Implementations/ (BaseRepository + 6个专用实现)
│   │   └── Extensions/ (LINQ扩展方法)
│   │
│   ├── Data/                                # 数据层
│   │   ├── ApplicationDbContext.cs
│   │   ├── Configurations/ (6个Fluent API配置)
│   │   ├── Migrations/
│   │   └── SeedData.cs
│   │
│   ├── Middleware/ (4个自定义中间件)
│   ├── Filters/ (3个过滤器)
│   ├── Helpers/ (8个辅助工具类)
│   │   ├── JwtHelper / SlugHelper / MarkdownService
│   │   ├── ArticleStatusMachine / CommentTreeBuilder
│   │   ├── TagCloudGenerator / ReadingTimeCalculator
│   │   └── ImageValidator
│   ├── Configuration/ (6个强类型配置类)
│   └── Extensions/ (DI/中间件/应用构建扩展)
│
├── BlogSystem.Frontend/                     # 🎨 Vue.js 3 前端项目
│   ├── src/
│   │   ├── api/                             # Axios封装 + 7个API模块
│   │   ├── stores/ (Pinia: auth/articles/comments/ui)
│   │   ├── router/ (路由配置 + 守卫 + 4组路由)
│   │   ├── views/ (5组页面: home/auth/article/category/search)
│   │   ├── components/ (20+组件: common/article/comment/editor/search)
│   │   ├── composables/ (useAuth/usePagination/useDebounce等)
│   │   ├── utils/ (format/storage/validators/constants)
│   │   ├── styles/ (CSS变量/global/markdown/highlight/transitions)
│   │   └── assets/
│   ├── package.json / vite.config.ts / tsconfig.json
│   └── .env.development / .env.production
│
├── BlogSystem.Tests/                        # 🧪 测试项目
│   ├── UnitTests/ (Services/Helpers/Middleware)
│   ├── IntegrationTests/ (Controllers/Fixtures)
│   └── Helpers/ (TestDataGenerator/MockHelpers)
│
├── deploy/                                  # 🚀 部署
│   ├── docker/ (Dockerfile.api/Dockerfile.frontend/docker-compose.yml)
│   ├── nginx/ (nginx.conf/SSL配置)
│   ├── scripts/ (deploy.sh/deploy.ps1/init-db.sql/backup-db.sh)
│   └── kubernetes/ (可选K8s部署配置)
│
├── docs/                                    # 📚 项目文档
├── .gitignore
├── README.md                                # 本文件
└── LICENSE
```

---

## 运行步骤

### 环境要求

#### 必须安装
| 工具 | 最低版本 | 安装方式 |
|------|---------|---------|
| .NET 8.0 SDK | 8.0.400+ | [dotnet.microsoft.com/download/dotnet/8.0](https://dotnet.microsoft.com/download/dotnet/8.0) |
| Node.js | 18 LTS+ | [nodejs.org](https://nodejs.org/) |
| SQL Server | 2019+ / LocalDB | [SQL Server Developer](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) |
| Redis | 7.0+ | [redis.io](https://redis.io/download)（Windows用Memurai或WSL2） |
| Git | 2.x | [git-scm.com](https://git-scm.com/) |
| Visual Studio 2022 / VS Code | 最新版 | IDE |

#### 可选（增强体验）
Docker Desktop（一键启动Redis/MinIO/Elasticsearch）、MinIO、Elasticsearch 8.x、Postman、SSMS、RedisInsight

### 步骤 1：克隆并还原依赖（5分钟）

```bash
git clone https://github.com/your-org/blog-system.git
cd blog-system

# 还原后端
dotnet restore BlogSystem.Api/BlogSystem.Api.csproj
dotnet restore BlogSystem.Tests/BlogSystem.Tests.csproj

# 还原前端
cd BlogSystem.Frontend && npm install   # 或 pnpm install
```

### 步骤 2：配置环境变量（5分钟）

**后端 appsettings.Development.json 关键配置**：

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BlogSystemDb;Trusted_Connection=True;"
  },
  "JwtSettings": {
    "SecretKey": "Your-Super-Secret-Key-Must-Be-At-Least-32-Characters!!!",
    "Issuer": "https://localhost:7000",
    "Audience": "https://localhost:5173",
    "AccessTokenExpirationMinutes": 15,
    "RefreshTokenExpirationDays": 7
  },
  "RedisSettings": { "ConnectionString": "localhost:6379", "InstanceName": "blog:" },
  "MinioSettings": { "Endpoint": "localhost:9000", "AccessKey": "minioadmin", "SecretKey": "minioadmin", "BucketName": "blog-images" },
  "UploadSettings": { "MaxFileSizeMB": 10, "PermittedExtensions": [".jpg",".jpeg",".png",".gif",".webp"] },
  "CorsSettings": { "AllowedOrigins": ["http://localhost:5173"] }
}
```

**前端 .env.development**：
```bash
VITE_API_BASE_URL=http://localhost:7000/api
VITE_APP_TITLE=我的技术博客
```

### 步骤 3：启动基础设施（5分钟）

**推荐：Docker Compose 一键启动**
```bash
cd deploy/docker
docker-compose up -d redis minio elasticsearch sqlserver
docker-compose ps   # 查看状态
```

基础设施包括：Redis（缓存）、MinIO（对象存储9000端口+Console 9001端口）、Elasticsearch（9200端口）、SQL Server（1433端口）。每个服务都配置了健康检查和数据持久化卷。

### 步骤 4：初始化数据库（3分钟）

```bash
cd BlogSystem.Api
dotnet ef migrations add InitialCreate    # 生成迁移
dotnet ef database update                  # 应用迁移（创建9张表）
# 验证：sqlcmd -S (localdb)\mssqllocaldb -d BlogSystemDb -Q "SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES"
```

预期表：__EFMigrationsHistory、AspNetUsers、Articles、ArticleVersions、Categories、Tags、ArticleTag、Comments、RefreshTokens

### 步骤 5：初始化 MinIO 存储桶（2分钟）

```bash
# 方式1：浏览器打开 http://localhost:9001（MinIO Console），登录 minioadmin/minioadmin，创建 "blog-images" Bucket
# 方式2：mc alias set myminio http://localhost:9000 minioadmin minioadmin && mc mb myminio/blog-images
```

### 步骤 6：启动后端 API（2分钟）

```bash
cd BlogSystem.Api
dotnet run   # 默认 http://localhost:7000
# 验证：浏览器打开 http://localhost:7000/swagger （Swagger UI）
```

### 步骤 7：启动前端（2分钟）

```bash
cd BlogSystem.Frontend
npm run dev   # http://localhost:5173
```

### 步骤 8：端到端验证（10分钟）

- [ ] POST /api/auth/register → 注册成功
- [ ] POST /api/auth/login → 返回 AccessToken + RefreshToken
- [ ] GET /api/users/me (Bearer Token) → 返回用户资料
- [ ] POST /api/categories → 创建分类（需Admin）
- [ ] POST /api/articles → 写文章（Markdown自动转HTML）
- [ ] GET /api/articles/{id} → 查看文章详情
- [ ] POST /api/articles/{id}/comments → 发表评论
- [ ] GET /api/articles/{id}/comments → 查看嵌套评论树
- [ ] POST /api/upload/images → 上传图片（返回WebP URL）
- [ ] GET /api/search?q=关键词 → 高亮搜索结果
- [ ] POST /api/auth/logout → Token被撤销

---

## 学习要点（25个知识点）

### 1. RESTful API 设计规范 ⭐⭐⭐⭐
**重要性**：★★★★★ | **教程**：[[进阶篇/01-RESTful-API设计]]

- URL 代表资源（名词），HTTP 方法代表操作（动词）
- 标准状态码：200 OK / 201 Created / 204 No Content / 400 Bad Request / 401 Unauthorized / 403 Forbidden / 404 Not Found / 409 Conflict / 429 Too Many Requests / 500 Internal Server Error
- 统一 ApiResponse\<T\> 包装格式
- 分页、排序、过滤的 QueryString 约定

### 2. JWT 认证与授权体系 ⭐⭐⭐⭐
**重要性**：★★★★★ | **教程**：[[进阶篇/02-JWT认证详解]]

完整流程：注册(bcrypt哈希) → 登录(验证凭据→生成JWT+RefreshToken) → 请求(Bearer Token→Middleware验证Claims) → 刷新(Token轮换) → 登出(黑名单)。Access Token 15分钟短有效期 + Refresh Token 7天HttpOnly Cookie + SameSite=Strict。

### 3. Entity Framework Core 高级用法 ⭐⭐⭐⭐
**重要性**：★★★★★ | **教程**：[[进阶篇/04-EF-Core高级特性]]

多对多关系(Article↔Tag跳表)、自引用(Comment嵌套/Category父子)、全局查询过滤器(软删除自动排除)、SplitQuery(防止Cartesian Explosion)、FromSqlRaw(全文搜索原始SQL)、ExecuteUpdateAsync/ExecuteDeleteAsync(批量操作EF Core 7+)。

### 4. Markdown 富文本处理 ⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/05-Markdown与富文本]]

Markdig Pipeline配置（AdvancedExtensions/AutoLinks/TaskLists/EmphasisExtras/PipeTables/SyntaxHighlighting）→ HTML输出 → XSS防护(HtmlSanitizer白名单: 允许p/br/code/pre/a/img/table等40个标签) → 后处理(代码块hljs类名/外部链接target="_blank"/图片lazy-loading)。

### 5. 嵌套评论树形构建算法 ⭐⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/06-树形数据处理]]

三种方案对比：内存构建O(n)(本项目采用，适合<2000评论/页) vs CTE递归查询(SQL Server/PostgreSQL原生支持) vs 物化路径Closure Table(百万级评论)。算法步骤：建ID→节点字典 → 遍历扁平列表构建父子关系 → 排序(根倒序/子正序) → 限制最大深度5层。

### 6. Redis 缓存策略 ⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/07-Redis缓存实践]]

10种使用场景：热门文章(Sorted Set 5min TTL)、文章详情(String JSON 10min)、标签云(String 1h)、热门搜索(Sorted Set 24h)、搜索历史(List 7天)、速率限制(String计数器)、在线用户(Set 5min心跳)、评论树(String JSON 2min)、Token黑名单(Set按TTL)、阅读量计数器(INCR 5min批量持久化)。Cache-Aside Pattern实现。

### 7. 前后端分离架构与 Vue.js 3 集成 ⭐⭐⭐⭐
**重要性**：★★★★★ | **教程**：[[进阶篇/08-Vue.js前端集成]] / [[进阶篇/09-前后端分离实战]]

架构：Browser(Vue3 SPA: Router+Pinia+Axios) → HTTPS → Nginx(反向代理/静态资源/SSL/Gzip) → ASP.NET Core Web API(JWT/RESTful/业务逻辑) → SQL Server+Redis+MinIO+Elasticsearch。Axios封装：请求拦截器(附加Bearer Token) + 响应拦截器(统一错误处理 + 401时自动刷新Token重试)。

### 8. 统一响应包装与全局异常处理 ⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/01-全局异常处理]]

ResponseWrapperMiddleware自动包装成功JSON响应；ExceptionMiddleware按异常类型返回不同状态码(NotFound→404/Validation→400/Conflict→409/Unauthorized→401/Forbidden→403/Internal→500)；开发环境返回StackTrace，生产环境隐藏敏感信息。

### 9. Swagger/OpenAPI 文档自动化 ⭐⭐
**重要性**：★★★★☆

Swashbuckle 配置：JWT Bearer 认证支持、XML 注释生成（启用 `<GenerateDocumentationFile>true`）、分组 API（按 Controller 分类）、示例值定义、Response 类型描述。访问 http://localhost:7000/swagger 交互式测试所有 API。

### 10. FluentValidation 高级验证 ⭐⭐⭐
**重要性**：★★★★☆

验证器类继承 AbstractValidator\<T\>，支持 RuleFor() 链式调用、自定义验证器（Must()）、条件验证（When()/Unless()）、跨属性验证（Custom()）、异步验证（MustAsync()）。与 ModelState 自动集成，错误消息国际化支持。

### 11. AutoMapper 复杂映射 ⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/02-DTO与AutoMapper]]

Profile 配置：Entity→DTO 投影（排除敏感字段如 PasswordHash/RowVersion）、枚举→字符串转换（Status→StatusName）、嵌套对象展开（Author→AuthorDto）、集合映射（Tags→List\<TagDto\>）、自定义解析（ResolveUsing()计算ReadingTime/生成AvatarUrl）、ReverseMap 支持 DTO→Entity 反向映射。

### 12. 速率限制与反滥用 ⭐⭐⭐
**重要性**：★★★★☆

AspRateLimit 固定窗口算法：登录(5次/IP/分)、注册(3次/IP/分)、评论(3次/用户/分)、上传(10次/用户/分)、搜索(30次/IP/分)。超出返回 429 + Retry-After 头。Redis 分布式计数器保证多实例一致性。

### 13. 日志系统（Serilog 结构化日志）⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/04-日志与监控]]

Serilog 配置：Console + File + Elasticsearch Sink；结构化模板（"用户 {UserId} 创建了文章 {ArticleId}: {Title}"）；日志级别规范（Trace/Debug/Info/Warning/Error/Critical）；请求日志中间件记录 Method/Path/StatusCode/Duration/UserId/TraceId。

### 14. 图片处理流水线 ⭐⭐⭐
**重要性**：★★★★☆

ImageSharp 库：按 UploadType 调整尺寸（封面OG 1200x630 / 内容1020px宽 / 头像256x256裁剪）→ 生成150x150缩略图 → WebP 编码（quality 85-90）→ MinIO S3 上传 → CDN URL 返回。三重验证（客户端JS + 服务端C# + Magic Number）。

### 15. 文章状态机模式 ⭐⭐⭐⭐
**重要性**：★★★★☆

ArticleStatusMachine 类：Dictionary\<Status, HashSet\<Status\>\> 定义合法转换矩阵；CanTransition() 静态验证方法；ValidateTransition() 抛出 InvalidOperationException；Service 层在 Update/Publish/Archive 操作前调用验证。

### 16. Elasticsearch 全文搜索 ⭐⭐⭐⭐
**重要性**：★★★★☆

NEST Client 配置：MultiMatch 多字段（标题权重3x/摘要2x/内容1x）+ Fuzziness.Auto 模糊匹配 + Bool 查询（Must匹配 + Filter 过滤）+ Highlight 高亮（pre/post tags mark标签）+ Sort（相关性优先+时间其次）。IndexPrefix 多环境隔离（blog_dev_/blog_prod_）。

### 17. 种子数据与数据库初始化 ⭐⭐
**重要性**：★★★☆☆

SeedData 类：开发环境自动插入示例数据（3个用户/5个分类/20个标签/15篇文章/50条评论）；HasData() 方法配合 Migration 使用；环境判断（Development 执行，Production 跳过）；Idempotent 设计（重复执行不报错）。

### 18. CORS 跨域配置 ⭐⭐
**重要性**：★★★★☆

AllowOrigins 白名单（localhost:5173 / 生产域名）；AllowCredentials（携带Cookie/Authorization头）；SetPreflightMaxAge(1h) 缓存 OPTIONS 预检请求；UseCors() 必须在 UseAuthentication() 之前注册。

### 19. Docker 容器化与编排 ⭐⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[高手篇/03-Docker容器化部署]]

多阶段构建 Dockerfile（SDK 构建 → Runtime 运行镜像，减小 80%+ 体积）；docker-compose.yml 编排 6 个服务（frontend/api/db/redis/minio/elasticsearch）；健康检查（HEALTHCHECK curl /api/health）；数据持久化（named volumes）；环境变量注入（.env 文件或 Docker Secrets）。

### 20. Nginx 反向代理与 SSL 终结 ⭐⭐⭐
**重要性**：★★★★☆

Nginx 配置：location /proxy_pass api:7000（API转发）；location /try_files $uri $uri/ /index.html（Vue Router history模式）；SSL 证书（Let's Encrypt Certbot 自动续期）；Gzip/Brotli 压缩；安全响应头（HSTS/X-Frame-Options/CSP/XSS-Protection/Referrer-Policy）。

### 21. 版本控制与 API 演进策略 ⭐⭐⭐
**重要性**：★★★★☆

URL 版本控制（/api/v1/articles）；向后兼容性（新字段 optional、不删除旧字段）；废弃警告（Deprecated Response Header）；Changelog 维护（CHANGELOG.md 记录 breaking changes）；Semantic Versioning（MAJOR.MINOR.PATCH）。

### 22. 单元测试与集成测试 ⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[进阶篇/06-单元测试最佳实践]]

单元测试（xUnit + Moq）：Service 层 AAA 模式（Arrange-Act-Assert）；Mock Repository 返回测试数据；边界条件测试（空输入/边界值/异常路径）；目标覆盖率 Service≥90%/Controller≥80%。集成测试（WebApplicationFactory）：真实 HTTP 请求/响应测试；TestDatabaseFixture 使用 SQLite InMemory 或 SQL Server LocalDB。

### 23. 配置管理（Options Pattern）⭐⭐
**重要性**：★★★★☆ | **教程**：[[基础篇/08-配置管理]]

强类型配置类（JwtSettings/RedisSettings/MinioSettings/ElasticsearchSettings）；builder.Services.Configure\<T\>() 注册；IOptions\<T\>/IOptionsMonitor\<T\>/IOptionsSnapshot\<T\> 注入；多环境覆盖（appsettings.{Environment}.json > 环境变量 > 命令行参数）；敏感配置外部化（Azure Key Vault / Docker Secrets / User Secrets）。

### 24. 异步编程最佳实践 ⭐⭐⭐
**重要性**：★★★★★ | **教程**：[[基础篇/07-异步编程详解]]

全程 async/await（禁止 .Result/.Wait() 死锁风险）；ConfigureAwait(false) 在库代码中使用；CancellationToken 支持（长耗时操作/导出/搜索）；ValueTask 优化同步快速路径；并行 async（Task.WhenAll 批量操作）。

### 25. 性能优化综合实践 ⭐⭐⭐⭐
**重要性**：★★★★☆ | **教程**：[[高手篇/01-性能优化指南]]

查询优化：AsNoTracking（只读+30-50%）/ Select投影（减传输60-80%）/ Include预加载（防N+1）/ SplitQuery（防笛卡尔积）；缓存策略：Redis 多层级缓存（热点数据/列表/详情/统计）；响应压缩（Brotli>Gzip，减小60-80%）；连接池（EF Core默认）；索引优化（复合索引/覆盖索引）；前端优化（懒加载/虚拟滚动/代码分割/CDN）。

---

## 性能指标参考

| 指标 | 目标值 | 测试条件 |
|------|--------|---------|
| API 平均响应时间（P50） | < 50ms | 简单 CRUD（缓存命中） |
| API 平均响应时间（P99） | < 500ms | 复杂查询（搜索/列表） |
| 首页加载时间 | < 1.5s | Chrome Lighthouse（4G网络） |
| 并发用户支持 | 500+ | 单服务器（4核8G） |
| 文章总数 | 10万+ | 数据库查询性能不显著下降 |
| 评论嵌套查询 | < 100ms | 单文章500条评论以