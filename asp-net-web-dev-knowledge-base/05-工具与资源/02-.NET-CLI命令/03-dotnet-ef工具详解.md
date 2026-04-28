# dotnet ef 工具详解

Entity Framework Core (EF Core) 是 .NET 生态中最流行的 ORM（对象关系映射）框架，而 `dotnet ef` 是其官方命令行工具。本文将全面介绍 `dotnet ef` 的安装、配置、迁移管理以及各种高级用法。

## 一、dotnet ef 安装与版本管理

### 1.1 安装方式对比

#### 方式一：全局工具安装（推荐用于开发环境）

```bash
# 安装最新版本
dotnet tool install --global dotnet-ef

# 安装指定版本（需与项目使用的 EF Core 版本匹配）
dotnet tool install --global dotnet-ef --version 8.0.10

# 更新到最新版本
dotnet tool update --global dotnet-ef

# 卸载全局工具
dotnet tool uninstall --global dotnet-ef

# 查看已安装的全局工具
dotnet tool list --global
```

**优点**：
- 所有项目共享同一版本
- 无需为每个项目重复安装
- 命令在任何目录下可用

**缺点**：
- 可能与其他项目的 EF Core 版本冲突
- 需要手动保持版本同步

#### 方式二：本地工具安装（推荐用于团队协作）

```bash
# 在项目根目录初始化本地工具清单
dotnet new tool-manifest

# 安装到当前项目
dotnet tool install dotnet-ef

# 安装指定版本
dotnet tool install dotnet-ef --version 8.0.10

# 查看已安装的本地工具
dotnet tool list

# 从其他目录运行（使用 --tool-path）
dotnet ef migrations list --tool-path ./tools
```

**manifest.json 文件内容示例**：

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "dotnet-ef": {
      "version": "8.0.10",
      "commands": [
        "dotnet-ef"
      ]
    }
  }
}
```

**优点**：
- 版本锁定在项目中，团队一致性高
- 通过源码管理自动共享配置
- 不同项目可以使用不同版本

### 1.2 版本匹配原则

```
┌─────────────────────────────────────────┐
│         版本匹配规则                      │
├─────────────────────────────────────────┤
│                                         │
│   .NET SDK 8.x  ─────►  EF Core 8.x     │
│   .NET SDK 9.x  ─────►  EF Core 9.x     │
│   .NET SDK 7.x  ─────►  EF Core 7.x     │
│                                         │
│   ⚠️ 重要：                              │
│   dotnet-ef 工具版本必须 ≥               │
│   项目引用的 Microsoft.EntityFrameworkCore 包版本 │
│                                         │
└─────────────────────────────────────────┘
```

**检查版本兼容性**：

```bash
# 查看 dotnet-ef 版本
dotnet ef --version

# 查看项目使用的 EF Core 版本
dotnet list package --include-transitive | findstr EntityFrameworkCore

# 或查看 csproj 文件中的包引用
grep -r "EntityFrameworkCore" *.csproj
```

### 1.3 常见安装问题解决

#### 问题：命令未找到

```bash
# 确认工具是否在 PATH 中
where dotnet-ef

# 如果找不到，添加到 PATH（Windows PowerShell）
$env:PATH += ";$env:USERPROFILE\.dotnet\tools"

# 或使用完整路径调用
~/.dotnet/tools/dotnet-ef.exe --version
```

#### 问题：版本不匹配错误

```bash
# 错误信息示例：
# "Cannot execute. The required command ... was not found."

# 解决方案：升级或降级到匹配的版本
dotnet tool update --global dotnet-ef --version 8.0.10
```

---

## 二、数据库迁移命令完整列表

### 2.1 迁移生命周期概览

```
┌──────────────────────────────────────────────────────────────┐
│                    迁移工作流程                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  [代码修改]                                                   │
│       │                                                      │
│       ▼                                                      │
│  dotnet ef migrations add <name>                             │
│       │                                                      │
│       ▼                                                      │
│  [生成迁移文件: Up/Down]                                      │
│       │                                                      │
│       ▼                                                      │
│  dotnet ef database update                                   │
│  或                                                          │
│  dotnet ef migrations script                                 │
│       │                                                      │
│       ▼                                                      │
│  [数据库结构更新]                                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 migrations add — 创建迁移

#### 基本语法

```bash
dotnet ef migrations add <迁移名称> [选项]
```

#### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--project` | 指定 DbContext 所在项目 | `-p src/Data` |
| `--startup-project` | 指定启动项目 | `-s src/Web` |
| `--context` | 指定 DbContext 名称 | `--context AppDbContext` |
| `--output-dir` | 迁移文件输出目录 | `--output-dir Data/Migrations` |
| `--namespace` | 迁移类命名空间 | `--namespace MyApp.Migrations` |

#### 命名规范最佳实践

```bash
# ✅ 推荐的命名方式（清晰描述变更）

# 添加新实体/表
dotnet ef migrations add CreateProductTable
dotnet ef migrations add AddOrderAndOrderItems

# 添加列
dotnet ef migrations add AddEmailColumnToUsers
dotnet ef migrations add AddIndexOnProductName

# 修改列
dotnet ef migrations add ChangePriceColumnTypeToDecimal
dotnet ef migrations add MakeTitleRequired

# 删除列/表
dotnet ef migrations remove  # 使用 remove 而非 add
dotnet ef migrations add RemoveLegacyTable

# 复杂变更
dotnet ef migrations add RefactorUserAuthSystem
dotnet ef migrations add SplitAddressIntoSeparateColumns

# ❌ 不推荐的命名
dotnet ef migrations add migration1        # 无意义
dotnet ef migrations add fix               # 过于模糊
dotnet ef migrations add update            # 不具体
```

#### 实际使用示例

```bash
# 场景一：标准 Web API 项目结构
cd MyProject
dotnet ef migrations add InitialCreate \
  --project src/MyProject.Infrastructure \
  --startup-project src/MyProject.Api \
  --context ApplicationDbContext

# 场景二：多 DbContext 项目
dotnet ef migrations add InitIdentityDb \
  --project src/Data \
  --context IdentityDbContext \
  --output-dir Migrations/Identity

dotnet ef migrations add InitAppDb \
  --project src/Data \
  --context AppDbContext \
  --output-dir Migrations/App

# 场景三：指定输出目录和命名空间
dotnet ef migrations add AddAuditTables \
  --output-dir Persistence/Migrations \
  --namespace MyProject.Persistence.Migrations
```

### 2.3 migrations list — 列出所有迁移

```bash
# 列出所有迁移（按时间顺序）
dotnet ef migrations list

# 仅列出待应用的迁移
dotnet ef migrations list --connection "Server=localhost;Database=mydb;..."

# 显示已应用的迁移
dotnet ef migrations list --applied-migrations-only

# 连接特定上下文
dotnet ef migrations list --context AppDbContext
```

**输出示例**：

```
20240101000000_InitialCreate
20240115000000_AddUserTable
20240201000000_AddOrderTable
20240215000000_AddIndexes
(pending) 20240301000000_AddAuditLogTable
```

### 2.4 migrations remove — 移除迁移

```bash
# 移除最近一次迁移（未应用到数据库时）
dotnet ef migrations remove

# 强制移除（即使已应用，慎用！）
dotnet ef migrations remove --force

# 指定项目
dotnet ef migrations remove -p src/Data
```

**注意事项**：
- 只能移除最后一个迁移
- 如果迁移已应用到数据库，需要先回滚或使用 `--force`
- `--force` 会删除数据库中的迁移记录，可能导致数据丢失

### 2.5 migrations script — 生成 SQL 脚本

```bash
# 生成从零开始的完整 SQL 脚本
dotnet ef migrations script

# 生成从指定迁移到当前的增量脚本
dotnet ef migrations script InitialCreate AddAuditLogTable

# 生成两个迁移之间的脚本
dotnet ef migrations script FromMigration ToMigration

# 输出到文件
dotnet ef migrations script -o ./scripts/migration.sql

# 幂等脚本（可用于任何状态的数据库）
dotnet ef migrations script --idempotent

# 不包含事务语句
dotnet ef migrations script --no-transactions
```

**生产环境部署推荐**：

```bash
# 生成幂等 SQL 脚本（安全）
dotnet ef migrations script --idempotent -o deploy/migration.sql

# 验证脚本内容
cat deploy/migration.sql | head -50
```

### 2.6 database update — 应用迁移到数据库

```bash
# 应用所有待处理的迁移
dotnet ef database update

# 应用到指定的迁移
dotnet ef database update InitialCreate
dotnet ef database update 20240101000000_InitialCreate

# 回滚到之前的迁移
dotnet ef database update PreviousMigrationName

# 回滚到初始状态（空数据库）
dotnet ef database update 0

# 指定连接字符串
dotnet ef database update --connection "Server=(localdb)\\mssqllocaldb;Database=MyDb"

# 指定项目
dotnet ef database update -p src/Data -s src/Web
```

**常用场景速查**：

| 场景 | 命令 |
|------|------|
| 首次创建数据库 | `dotnet ef database update` |
| 应用最新迁移 | `dotnet ef database update` |
| 回滚一个版本 | `dotnet ef database update <上一个迁移名>` |
| 完全重置数据库 | `dotnet ef database update 0` |
| 指定目标数据库 | `dotnet ef database update --connection "..."` |

### 2.7 database drop — 删除数据库

```bash
# 删除数据库（会提示确认）
dotnet ef database drop

# 强制删除（无提示）
dotnet ef database drop --force

# 删除指定连接的数据库
dotnet ef database drop --connection "Server=..."
```

### 2.8 dbcontext 信息命令

```bash
# 显示 DbContext 类型信息
dotnet ef dbcontext info

# 显示 DbContext 的详细信息
dotnet ef dbcontext info --context AppDbContext

# 列出所有可用的 DbContext
dotnet ef dbcontext list

# 输出示例：
# Your project uses the following DbContext types:
# - MyProject.Data.AppDbContext (Microsoft.EntityFrameworkCore.SqlServer)
# - MyProject.Data.IdentityDbContext (Microsoft.EntityFrameworkCore.SqlServer)
```

### 2.9 dbcontext scaffold — 反向工程（数据库优先）

```bash
# 从现有数据库生成 DbContext 和实体类
dotnet ef dbcontext scaffold \
  "Server=(localdb)\\mssqllocaldb;Database=MyDb;Trusted_Connection=True;" \
  Microsoft.EntityFrameworkCore.SqlServer \
  -o Models \
  --context-dir Data \
  --context MyDbContext \
  --tables Products,Orders,Customers \
  --use-database-names \
  --data-annotations \
  --force
```

**常用参数说明**：

| 参数 | 说明 |
|------|------|
| `-o` / `--output-dir` | 实体模型输出目录 |
| `--context-dir` | DbContext 输出目录 |
| `--context` | DbContext 类名 |
| `--tables` | 指定要生成的表（逗号分隔） |
| `--schema` | 指定数据库架构 |
| `--use-database-names` | 使用数据库原始名称 |
| `--data-annotations` | 使用数据注解而非 Fluent API |
| `--force` | 覆盖已有文件 |
| `--no-onconfiguring` | 不生成 OnConfiguring 方法 |
| `--namespace` | 模型命名空间 |
| `--context-namespace` | DbContext 命名空间 |

**高级 Scaffold 配置**：

```bash
# 完整的企业级反向工程配置
dotnet ef dbcontext scaffold \
  "Server=myserver;Database=production;User Id=sa;Password=***;" \
  Microsoft.EntityFrameworkCore.SqlServer \
  --project src/MyProject.Data \
  --startup-project src/MyProject.Web \
  --output-dir Entities \
  --context-dir Contexts \
  --context ProductionDbContext \
  --tables Users,Roles,Permissions,UserRoles, \
           Products,Categories,ProductCategories, \
           Orders,OrderItems,Payments \
  --schema dbo \
  --use-database-names \
  --data-annotations \
  --force \
  --namespace MyProject.Domain.Entities \
  --context-namespace MyProject.Infrastructure.Contexts
```

---

## 三、常用场景速查

### 3.1 首次迁移完整流程

```bash
# 步骤 1：确保项目结构正确
# - 数据访问层项目包含 DbContext
# - 引用了 EF Core 相关 NuGet 包
# - 启动项目有正确的连接字符串配置

# 步骤 2：创建初始迁移
dotnet ef migrations add InitialCreate \
  --project src/MyProject.Infrastructure \
  --startup-project src/MyProject.Api

# 步骤 3：检查生成的迁移文件
ls src/MyProject.Infrastructure/Migrations/

# 步骤 4：预览将执行的 SQL
dotnet ef migrations script --idempotent | head -100

# 步骤 5：应用迁移到开发数据库
dotnet ef database update \
  --project src/MyProject.Infrastructure \
  --startup-project src/MyProject.Api

# 步骤 6：验证数据库
dotnet ef dbcontext info
```

### 3.2 回滚迁移

```bash
# 场景一：回滚到上一版本
dotnet ef database update PreviousMigrationName

# 场景二：回滚多个版本
dotnet ef database update TargetMigrationName

# 场景三：完全重置（开发环境）
dotnet ef database drop --force
dotnet ef database update

# 场景四：移除最后一次迁移并重新生成
dotnet ef migrations remove
# 修改模型...
dotnet ef migrations add FixedMigrationName
```

### 3.3 生产环境 SQL 脚本部署

```bash
# 生成生产就绪的 SQL 脚本
dotnet ef migrations script \
  --idempotent \
  --output ./deploy/$(date +%Y%m%d)_migration.sql \
  --project src/Infrastructure \
  --startup-project src/WebApi

# 脚本验证清单
echo "=== 部署前检查 ==="
echo "1. 检查脚本是否包含 BEGIN/COMMIT TRANSACTION"
echo "2. 检查是否有 DROP COLUMN 或 DROP TABLE 操作"
echo "3. 在测试数据库上执行一次"
echo "4. 备份生产数据库"
echo "5. 安排维护窗口"

# CI/CD 流水线集成示例
# .github/workflows/deploy.yml
# - name: Generate Migration Script
#   run: |
#     dotnet ef migrations script --idempotent -o deploy/migration.sql
#
# - name: Deploy to Production
#   run: |
#     sqlcmd -S prod-server -d production -i deploy/migration.sql
```

### 3.4 种子数据管理

#### 方式一：迁移中嵌入种子数据

```csharp
// Migrations/<timestamp>_SeedData.cs
public partial class SeedData : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // 插入种子数据
        migrationBuilder.InsertData(
            table: "Roles",
            columns: new[] { "Id", "Name", "NormalizedName" },
            values: new object[,]
            {
                { Guid.NewGuid(), "Admin", "ADMIN" },
                { Guid.NewGuid(), "User", "USER" },
                { Guid.NewGuid(), "Moderator", "MODERATOR" }
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // 回滚时清理种子数据
        migrationBuilder.DeleteData(
            table: "Roles",
            keyColumn: "NormalizedName",
            keyValues: new[] { "ADMIN", "USER", "MODERATOR" });
    }
}
```

#### 方式二：程序启动时初始化（推荐）

```csharp
// Data/Seeder.cs
public static class Seeder
{
    public async static Task SeedAsync(this IServiceProvider services)
    {
        using var scope = services.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        // 确保数据库已创建
        await context.Database.EnsureCreatedAsync();

        // 检查是否已有数据
        if (await context.Roles.AnyAsync()) return;

        // 添加角色
        var roles = new List<Role>
        {
            new() { Name = "Admin", NormalizedName = "ADMIN" },
            new() { Name = "User", NormalizedName = "USER" }
        };
        await context.Roles.AddRangeAsync(roles);
        await context.SaveChangesAsync();

        Console.WriteLine("✓ 种子数据初始化完成");
    }
}

// Program.cs
app.Services.SeedAsync().Wait();
```

#### 方式三：使用扩展方法（EF Core 7+）

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // 配置种子数据（不可变数据）
        modelBuilder.Entity<Role>().HasData(
            new Role { Id = 1, Name = "Admin", NormalizedName = "ADMIN" },
            new Role { Id = 2, Name = "User", NormalizedName = "USER" }
        );
    }
}

// 注意：HasData 要求显式设置主键值
// 适用于参考数据（如枚举映射），不适合动态数据
```

### 3.5 多 DbContext 管理

#### 项目结构示例

```
src/
├── MyProject.Data/
│   ├── Contexts/
│   │   ├── AppDbContext.cs          # 业务数据
│   │   ├── IdentityDbContext.cs     # 身份数据
│   │   └── LoggingDbContext.cs      # 日志数据
│   └── Migrations/
│       ├── App/                     # AppDbContext 迁移
│       │   ├── 20240101_InitialCreate.cs
│       │   └── AppDbContextModelSnapshot.cs
│       ├── Identity/                # IdentityDbContext 迁移
│       │   ├── 20240101_CreateIdentitySchema.cs
│       │   └── IdentityDbContextModelSnapshot.cs
│       └── Logging/                 # LoggingDbContext 迁移
│           ├── 20240101_CreateLoggingTables.cs
│           └── LoggingDbContextModelSnapshot.cs
```

#### 多 DbContext 迁移命令

```bash
# 为每个 DbContext 单独管理迁移

# AppDbContext
dotnet ef migrations add InitApp \
  --context AppDbContext \
  --project src/Data \
  --output-dir Migrations/App

dotnet ef database update \
  --context AppDbContext \
  --project src/Data

# IdentityDbContext
dotnet ef migrations add InitIdentity \
  --context IdentityDbContext \
  --project src/Data \
  --output-dir Migrations/Identity

dotnet ef database update \
  --context IdentityDbContext \
  --project src/Data

# 批量更新所有 DbContext
for ctx in AppDbContext IdentityDbContext LoggingDbContext; do
  echo "Updating $ctx..."
  dotnet ef database update --context $ctx --project src/Data
done
```

---

## 四、迁移文件深度解读

### 4.1 迁移文件结构

每次执行 `migrations add` 会生成以下文件：

```
Migrations/
├── 20240315120000_AddUserTable.cs      # 迁移主文件（Up/Down 方法）
├── 20240315120000_AddUserTable.Designer.cs  # 元数据（自动生成）
└── AppDbContextModelSnapshot.cs        # 当前模型快照
```

### 4.2 迁移主文件解析

```csharp
using Microsoft.EntityFrameworkCore.Migrations;
using Microsoft.EntityFrameworkCore.Metadata;

#nullable disable

namespace MyProject.Data.Migrations
{
    /// <summary>
    /// 迁移文件：添加用户表
    /// </summary>
    public partial class AddUserTable : Migration
    {
        /// <summary>
        /// Up 方法：应用此迁移时执行的操作（向前迁移）
        /// </summary>
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            // 创建新表
            migrationBuilder.CreateTable(
                name: "Users",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1,1"),
                    Email = table.Column<string>(type: "nvarchar(256)", maxLength: 256, nullable: false),
                    UserName = table.Column<string>(type: "nvarchar(256)", maxLength: 256, nullable: false),
                    CreatedAt = table.Column<DateTime>(type: "datetime2", nullable: false),
                    IsActive = table.Column<bool>(type: "bit", nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Users", x => x.Id);
                });

            // 创建索引
            migrationBuilder.CreateIndex(
                name: "IX_Users_Email",
                table: "Users",
                column: "Email",
                unique: true);

            // 添加注释（SQL Server 2017+）
            migrationBuilder.AddSql(
                @"EXEC sp_addextendedproperty 
                  @name = N'MS_Description', 
                  @value = N'用户表', 
                  @level0type = N'SCHEMA', @level0name = N'dbo',
                  @level1type = N'TABLE',  @level1name = N'Users'");
        }

        /// <summary>
        /// Down 方法：回滚此迁移时执行的操作（向后迁移）
        /// </summary>
        protected override void Down(MigrationBuilder migrationBuilder)
        {
            // 按 Up 的逆序操作
            migrationBuilder.DropTable(name: "Users");
        }
    }
}
```

### 4.3 常见迁移操作对照表

| C# 模型变更 | Up 方法生成的操作 | Down 方法生成的操作 |
|-------------|-------------------|---------------------|
| 新增实体类 | `CreateTable()` | `DropTable()` |
| 新增属性 | `AddColumn()` | `RemoveColumn()` |
| 删除属性 | `RemoveColumn()` | `AddColumn()` |
| 修改属性类型 | `AlterColumn()` | `AlterColumn()` |
| 添加 `[Required]` | `AlterColumn(nullable: false)` | `AlterColumn(nullable: true)` |
| 添加索引 | `CreateIndex()` | `DropIndex()` |
| 添加外键 | `AddForeignKey()` | `DropForeignKey()` |
| 修改主键 | `DropPrimaryKey()` + `AddPrimaryKey()` | 反向操作 |
| 重命名列 | `RenameColumn()` | `RenameColumn()` |
| 重命名表 | `RenameTable()` | `RenameTable()` |

### 4.4 自定义迁移操作

有时需要执行 EF Core 无法自动检测的操作：

```csharp
public partial class CustomMigration : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // 执行原生 SQL
        migrationBuilder.Sql(@"
            CREATE TRIGGER [dbo].[UpdateTimestamp]
            ON [dbo].[Entities]
            AFTER UPDATE
            AS
            BEGIN
                SET NOCOUNT ON;
                UPDATE [dbo].[Entities]
                SET [UpdatedAt] = GETUTCDATE()
                FROM INSERTED i
                WHERE [dbo].[Entities].[Id] = i.[Id];
            END
        ");

        // 创建存储过程
        migrationBuilder.Sql(@"
            CREATE PROCEDURE [dbo].[GetTopProducts]
                @CategoryId INT,
                @Take INT = 10
            AS
            BEGIN
                SELECT TOP (@Take) *
                FROM Products
                WHERE CategoryId = @CategoryId
                ORDER BY CreatedAt DESC;
            END
        ");

        // 插入初始数据
        migrationBuilder.InsertData(
            table: "Settings",
            columns: new[] { "Key", "Value", "Description" },
            values: new object[,]
            {
                { "SiteName", "My App", "网站名称" },
                { "Version", "1.0.0", "系统版本" }
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // 清理自定义对象
        migrationBuilder.Sql("DROP PROCEDURE IF EXISTS [dbo].[GetTopProducts]");
        migrationBuilder.Sql("DROP TRIGGER IF EXISTS [dbo].[UpdateTimestamp]");

        // 清理种子数据
        migrationBuilder.DeleteData(
            table: "Settings",
            keyColumn: "Key",
            keyValues: new object[] { "SiteName", "Version" });
    }
}
```

### 4.5 忽略模型变更

某些情况下需要排除特定模型的迁移检测：

```csharp
// 在 DbContext 中配置
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // 排除不需要迁移的实体
    modelBuilder.Ignore<ReadOnlyView>();
    modelBuilder.Ignore<TemporaryCache>();

    // 排除特定表的变更检测
    modelBuilder.Entity<AuditLog>().ToTable(t => t.ExcludeFromMigrations());
}
```

---

## 五、进阶技巧与最佳实践

### 5.1 迁移策略选择

| 策略 | 适用场景 | 优缺点 |
|------|----------|--------|
| **自动迁移** | 开发阶段原型 | 快速但难以控制 |
| **显式迁移**（推荐） | 生产环境 | 可控、可审计、可回滚 |
| **混合模式** | 大型团队 | 平衡效率与控制 |

### 5.2 迁移文件组织建议

```
Data/
├── Migrations/
│   ├── Common/
│   │   ├── 20240101_InitialSchema.cs
│   │   └── 20240102_AddIndexes.cs
│   ├── Features/
│   │   ├── Auth/
│   │   │   ├── 20240201_AddIdentityTables.cs
│   │   │   └── 20240202_AddTwoFactorAuth.cs
│   │   └── ECommerce/
│   │       ├── 20240301_AddProductCatalog.cs
│   │       └── 20240302_AddShoppingCart.cs
│   └── Hotfixes/
│       └── 20240401_FixOrderTotalCalculation.cs
└── ...
```

### 5.3 CI/CD 集成模板

```yaml
# .github/workflows/ef-migrations.yml
name: EF Core Migrations

on:
  push:
    branches: [main, develop]

jobs:
  check-migrations:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Install EF Core tools
        run: dotnet tool install --global dotnet-ef

      - name: Check for pending model changes
        run: |
          dotnet ef migrations has-pending-model-changes \
            --project src/Infrastructure \
            --startup-project src/WebApi || exit 1

  generate-script:
    needs: check-migrations
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Install EF Core tools
        run: dotnet tool install --global dotnet-ef

      - name: Generate migration script
        run: |
          dotnet ef migrations script --idempotent \
            -o deployment/migration-$(date +%Y%m%d-%H%M%S).sql \
            --project src/Infrastructure \
            --startup-project src/WebApi

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: migration-scripts
          path: deployment/*.sql
```

### 5.4 性能优化建议

1. **批量操作**：对于大量数据变更，考虑使用 `ExecuteUpdate` 和 `ExecuteDelete`（EF Core 7+）
2. **延迟加载注意**：避免 N+1 查询问题
3. **连接池配置**：合理设置连接池大小
4. **索引规划**：根据查询模式创建适当的索引
5. **迁移拆分**：大型变更拆分为多个小迁移，便于回滚

### 5.5 故障排查指南

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 迁移挂起 | 数据库被锁 | 检查活跃连接，重启服务 |
| 迁移冲突 | 多人同时修改模型 | 合并迁移文件或重新生成 |
| 脚本执行失败 | 权限不足 | 检查数据库用户权限 |
| 种子数据重复 | 重复执行种子方法 | 使用 `EnsureCreated()` 或检查存在性 |
| 性能下降 | 缺少索引或查询不当 | 分析执行计划，优化查询 |

---

## 六、命令速查卡

### 核心命令一览

```bash
# ===== 安装与管理 =====
dotnet tool install --global dotnet-ef              # 全局安装
dotnet tool install dotnet-ef                       # 本地安装
dotnet ef --version                                  # 查看版本

# ===== 迁移操作 =====
dotnet ef migrations add <name>                     # 创建迁移
dotnet ef migrations list                           # 列出迁移
dotnet ef migrations remove                         # 移除最后迁移
dotnet ef migrations script                         # 生成 SQL 脚本
dotnet ef migrations script --idempotent            # 幂等脚本

# ===== 数据库操作 =====
dotnet ef database update                           # 应用迁移
dotnet ef database update <migration>               # 应用到指定迁移
dotnet ef database update 0                          # 回滚到初始状态
dotnet ef database drop                             # 删除数据库
dotnet ef database drop --force                     # 强制删除

# ===== DbContext 操作 =====
dotnet ef dbcontext info                            # 显示上下文信息
dotnet ef dbcontext list                            # 列出所有上下文
dotnet ef dbcontext scaffold "<conn>" <provider>    # 反向工程

# ===== 常用选项 =====
--project <path>                                    # 指定项目路径
--startup-project <path>                            # 指定启动项目
--context <name>                                    # 指定 DbContext
--connection "<string>"                             # 连接字符串
-o / --output-dir <dir>                             # 输出目录
--verbose                                           # 详细输出
--dry-run                                           # 预览模式
```

掌握 `dotnet ef` 工具的使用，是进行高效 EF Core 开发的基础。通过合理运用迁移命令、遵循命名规范、结合 CI/CD 自动化流程，可以确保数据库 schema 变更的安全性、可追溯性和可回滚性。
