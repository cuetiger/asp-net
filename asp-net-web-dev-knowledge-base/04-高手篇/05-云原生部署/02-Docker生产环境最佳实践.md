# Docker生产环境最佳实践

> **标签**：#Docker #容器化 #生产环境 #安全加固
> **阅读时间**：约30分钟 | **难度**：⭐⭐⭐⭐
> **前置知识**：[[02-微服务架构/容器化Docker基础]]、[[01-设计模式/依赖注入进阶]]

---

## 目录

- [一、开发环境Docker vs 生产环境Docker的关键差异](#一开发环境docker-vs-生产环境docker的关键差异)
- [二、多阶段构建(Multi-stage Build)最佳实践](#二多阶段构建multi-stage-build最佳实践)
- [三、Dockerfile安全加固](#三dockerfile安全加固)
- [四、生产级docker-compose配置](#四生产级docker-compose配置)
- [五、容器编排健康检查](#五容器编排健康检查)
- [六、资源限制与请求](#六资源限制与请求)
- [七、日志驱动配置](#七日志驱动配置)
- [八、网络隔离策略](#八网络隔离策略)
- [九、Secrets敏感信息管理](#九secrets敏感信息管理)
- [十、完整生产级模板](#十完整生产级模板)

---

## 一、开发环境Docker vs 生产环境Docker的关键差异

### 1.1 核心差异对比

许多开发者在本地用Docker跑得很顺利，但一到生产环境就问题频出。这是因为开发和生产对Docker的使用有着本质区别：

```mermaid
graph TB
    subgraph "开发环境 Docker"
        D1[目标: 快速迭代] --> D2[镜像大一点没关系]
        D2 --> D3[包含调试工具]
        D3 --> D4[root用户运行]
        D4 --> D5[热重载启用]
        D5 --> D6[日志输出到控制台]
        D6 --> D7[无资源限制]
    end

    subgraph "生产环境 Docker"
        P1[目标: 稳定安全高效] --> P2[镜像尽可能小]
        P2 --> P3[只含运行时依赖]
        P3 --> P4[非root用户运行]
        P4 --> P5[优化启动速度]
        P5 --> P6[结构化日志+集中收集]
        P6 --> P7[严格的资源限制]
    end

    style D1 fill:#74c0fc,color:#333
    style P1 fill:#51cf66,color:#fff
```

| 维度 | 开发环境 | 生产环境 | 原因 |
|------|---------|---------|------|
| **镜像大小** | 1-2GB可接受 | <200MB为佳 | 减少攻击面、加快分发 |
| **基础镜像** | mcr.microsoft.com/dotnet/sdk | dotnet/aspnet精简版 | SDK不需要在运行时存在 |
| **用户权限** | root（方便调试） | 非root用户 | 安全合规要求 |
| **调试符号** | 包含pdb文件 | 移除所有调试信息 | 减小体积、保护代码 |
| **端口暴露** | 暴露所有端口 | 只暴露必要端口 | 减少攻击面 |
| **资源限制** | 无限制 | CPU/内存严格限制 | 防止资源耗尽影响其他容器 |
| **日志** | 控制台输出 | 结构化日志+外部收集 | 便于排查和审计 |
| **重启策略** | 不需要 | unless-stopped/restart: always | 保证高可用 |

### 1.2 常见的"开发转生产"陷阱

```mermaid
flowchart TD
    A[开发环境正常] --> B{生产环境出问题?}

    B --> C["❌ 镜像太大<br/>下载慢/存储成本高"]
    B --> D["❌ 以root运行<br/>安全扫描不通过"]
    B --> E["❌ 无健康检查<br/>K8s无法正确调度"]
    B --> F["❌ 时区错误<br/>日志时间混乱"]
    B --> G["❌ PID 1问题<br/>信号处理异常"]

    C --> H["解决方案:<br/>使用多阶段构建"]
    D --> I["解决方案:<br/>添加非root用户"]
    E --> J["解决方案:<br/>配置HEALTHCHECK"]
    F --> K["解决方案:<br/>设置TZ环境变量"]
    G --> L["解决方案:<br/>使用tini或dumb-init"]
```

---

## 二、多阶段构建(Multi-stage Build)最佳实践

### 2.1 为什么需要多阶段构建

传统的Dockerfile会将SDK、源代码、编译产物全部打包到最终镜像中，导致：
- 镜像体积巨大（通常1-2GB）
- 攻击面广（包含编译器等不必要的工具）
- 构建时间长（每次都要重新编译）
- 安全风险高（可能泄露源代码）

**多阶段构建的核心思想**：在一个或多个阶段完成编译，只将必要的运行时产物复制到最终镜像。

```mermaid
flowchart LR
    subgraph "阶段1: SDK构建"
        A1[dotnet/sdk:8.0] --> A2[dotnet restore]
        A2 --> A3[dotnet build]
        A3 --> A4[dotnet test]
        A4 --> A5[dotnet publish<br/>输出: /app/publish]
    end

    subgraph "阶段2: Runtime运行"
        B1[dotnet/aspnet:8.0] --> B2[COPY --from=build /app/publish .]
        B2 --> B3[ENTRYPOINT<br/>dotnet MyApi.dll]
    end

    A5 -->|"只复制publish产物"| B2

    style A1 fill:#fcc419,color:#333
    style B1 fill:#51cf66,color:#fff
```

### 2.2 完整的生产级多阶段Dockerfile

下面是一个经过实战验证的ASP.NET Core 8生产级Dockerfile，包含了所有最佳实践：

```dockerfile
# ============================================================
# ASP.NET Core 8 生产级 Dockerfile
# 特点: 多阶段构建 + 安全加固 + 最小化镜像
# ============================================================

# ==================== 阶段1: 构建阶段 ====================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release

# 设置工作目录
WORKDIR /src

# 先复制csproj和sln文件，利用Docker层缓存加速restore
COPY ["MyApi/MyApi.csproj", "MyApi/"]
COPY ["MyApi.Tests/MyApi.Tests.csproj", "MyApi.Tests/"]

# 还原NuGet包（这一层会被缓存）
RUN dotnet restore "MyApi/MyApi.csproj" \
    --configfile NuGet.Config \
    --runtime linux-x64

# 复制其余源代码
COPY . .

# 编译项目（不发布）
RUN dotnet build "MyApi/MyApi.csproj" \
    -c $BUILD_CONFIGURATION \
    --no-restore \
    /p:TreatWarningsAsErrors=true \
    /p:WarningsAsErrors=true

# 运行单元测试（测试失败则构建失败）
RUN dotnet test "MyApi.Tests/MyApi.Tests.csproj" \
    -c $BUILD_CONFIGURATION \
    --no-build \
    --verbosity normal \
    --logger:"trx;LogFileName=test-results.trx"

# 发布项目（自包含部署或依赖框架部署）
# 注意: 生产环境推荐使用依赖框架部署(FDD)，更小的镜像
RUN dotnet publish "MyApi/MyApi.csproj" \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    --self-contained false \
    -p:PublishTrimmed=false \
    -p:PublishSingleFile=false \
    /p:UseAppHost=false

# ==================== 阶段2: 运行时阶段（Alpine精简版）====================
FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime

# ===== 安全加固开始 =====

# 1. 安装必要的安全工具和时区数据
# Alpine使用apk包管理器
RUN apk add --no-cache \
    icu-libs \                    # 国际化支持（日期格式化等）
    icu-data-full \               # 完整的ICU数据
    tzdata \                      # 时区数据
    curl \                        # 用于健康检查
    tini \                        # PID 1初始化系统（正确处理信号）

# 2. 创建非root用户和组
# 安全最佳实践：绝不以root身份运行应用
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup -h /app -s /sbin/nologin

# 3. 创建必要的目录并设置正确的所有权
RUN mkdir -p /app/logs /app/temp && \
    chown -R appuser:appgroup /app /tmp

# 4. 设置环境变量
ENV ASPNETCORE_URLS=http://+:8080 \
    ASPNETCORE_ENVIRONMENT=Production \
    # 配置.NET运行时行为
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    DOTNET_RUNNING_IN_CONTAINER=true \
    TZ=Asia/Shanghai \
    # 性能优化
    DOTNET_GCHeapHardLimit=536870912 \           # 512MB堆内存硬限
    DOTNET_GCHighMemPercent=75 \                  # 高内存阈值百分比
    DOTNET_TieredPGO=1 \                          # 分层PGO优化
    # 日志配置
    Logging__Console__FormatterName=json \
    Logging__LogLevel__Default=Information \
    # 安全相关
    COMPlus_DbgEnableMiniDump=0 \                 # 禁用minidump防止信息泄露
    DOTNET_EnableDiagnostics=0                    # 禁用诊断端口

# 切换到非root用户
USER appuser

# 设置工作目录
WORKDIR /app

# 从构建阶段复制发布产物
# 使用--chown确保文件属于正确的用户
COPY --from=build --chown=appuser:appgroup /app/publish .

# 5. 健康检查配置
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl --fail http://localhost:8080/health || exit 1

# 6. 暴露端口
EXPOSE 8080

# 7. 使用tini作为PID 1进程
# 这确保了SIGTERM信号能正确传递给.NET应用
ENTRYPOINT ["tini", "--"]

# 启动应用
CMD ["dotnet", "MyApi.dll"]

# ===== 安全加固结束 =====
```

### 2.3 不同基础镜像的选择指南

.NET 8提供了多种运行时镜像，选择正确的镜像对最终大小有巨大影响：

```mermaid
graph TB
    subgraph "Ubuntu系列"
        U1["aspnet:8.0<br/>~230MB<br/>完整功能<br/>兼容性最好"] 
        U2["aspnet:8.0-jammy<br/>~220MB<br/>Ubuntu 22.04 LTS<br/>长期支持"]
        U3["aspnet:8.0-chiseled<br/>~180MB<br/>裁剪版Ubuntu<br/>安全更新好"]
    end

    subgraph "Debian系列"
        D1["aspnet:8.0<br/>~210MB<br/>默认Debian<br/>稳定可靠"]
        D2["aspnet:8.0-bookworm-slim<br/>~130MB<br/>精简Debian<br/>平衡之选"]
    end

    subgraph "Alpine系列"
        A1["aspnet:8.0-alpine<br/>~90MB<br/>最小体积<br/> musl libc"]
    end

    U1 -->|"如果需要完整兼容"| S[选型决策]
    U2 -->|"需要LTS支持"| S
    U3 -->|"关注安全+体积"| S
    D1 -->|"默认选择"| S
    D2 -->|"平衡体积与兼容"| S
    A1 -->|"追求极致小体积"| S

    style A1 fill:#51cf66,color:#fff
    style D2 fill:#fcc419,color:#333
    style U3 fill:#74c0fc,color:#333
```

**详细对比表**：

| 镜像标签 | 大小 | 包管理器 | libc | 适用场景 | 兼容性问题 |
|---------|------|---------|------|---------|-----------|
| `aspnet:8.0` (Debian) | ~210MB | apt | glibc | 默认推荐，最稳定 | 无 |
| `aspnet:8.0-bookworm-slim` | ~130MB | apt | glibc | **生产首选**，平衡性好 | 无 |
| `aspnet:8.0-alpine` | ~90MB | apk | musl | 追求极致小体积 | 部分原生库需重新编译 |
| `aspnet:8.0-chiseled` | ~180MB | 无(apt-get) | glibc | 安全敏感场景 | 某些工具缺失 |
| `aspnet:8.0-ubuntu-jammy` | ~220MB | apt | glibc | 需要Ubuntu生态 | 无 |

> **我的推荐**：对于大多数ASP.NET Core生产应用，使用 **`mcr.microsoft.com/dotnet/aspnet:8.0` 或 `bookworm-slim`** 是最稳妥的选择。只有在镜像仓库存储成本极其敏感时才考虑Alpine。

### 2.4 进一步减小镜像体积的技巧

#### 技巧1：使用.dockerignore文件

```dockerignore
# .dockerignore - 排除不需要进入镜像的文件
**/.git
**/.gitignore
**/.vs
**/.vscode
**/*.md
**/docs
**/tests
**/*.Tests
**/bin/Debug
**/obj/Debug
**/*.csproj.user
**/Directory.Build.props
**/NuGet.config
**/azure-pipelines.yml
**/.github
**/Dockerfile*
**/docker-compose*.yml
**/.env*
**/node_modules
**/wwwroot/lib          # 如果前端库通过CDN加载
**/*.pdb                # 调试符号文件
**/*.Dev.dll            # 开发时DLL
```

#### 技巧2：合并RUN指令减少层数

```dockerfile
# ❌ 错误做法：多个RUN指令产生多层
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# ✅ 正确做法：合并为一个RUN指令
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

#### 技巧3：合理排序指令以最大化缓存

```dockerfile
# ✅ 正确顺序：变化频率低的放前面
# 1. 基础镜像（很少变）
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime

# 2. 安装系统包（偶尔变）
RUN apk add --no-cache icu-libs tzdata curl tini

# 3. 创建用户（基本不变）
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# 4. 复制依赖文件（项目引用不变时不变）
COPY --from=build /app/publish MyApp.deps.json ./

# 5. 复制主程序（每次构建都变）
COPY --from=build /app/publish MyApp.dll ./

# 6. 复制其他文件（可能经常变）
COPY --from=build /app/publish appsettings.Production.json ./
```

---

## 三、Dockerfile安全加固

### 3.1 COPY --chown 安全文件权限

当从构建阶段复制文件时，始终指定正确的所有权：

```dockerfile
# ❌ 危险：以root身份复制文件
COPY --from=build /app/publish .

# ✅ 安全：指定非root用户作为所有者
COPY --from=build --chown=appuser:appgroup /app/publish .
```

**为什么重要**：即使你切换了USER，如果文件属于root，应用可能因为权限不足而无法写入日志或临时文件。

### 3.2 HEALTHCHECK 健康检查

健康检查是生产环境的必备配置，它让Docker和Kubernetes知道应用是否正常运行：

```dockerfile
# 基本健康检查
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:8080/health/live || exit 1

# 更完善的健康检查（区分存活和就绪）
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8080/health/ready || exit 1
```

**参数说明**：

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| `--interval` | 30s | 检查间隔，太频繁消耗资源 |
| `--timeout` | 5-10s | 超时时间，应小于interval |
| `--start-period` | 60-120s | 启动宽限期，给应用足够的启动时间 |
| `--retries` | 3 | 连续失败多少次标记为unhealthy |

### 3.3 TMPDIR 临时目录配置

.NET应用在运行时会创建临时文件（如编译视图、Session文件等），必须确保临时目录可写且安全：

```dockerfile
# 在USER切换之前配置
ENV TMPDIR=/app/temp \
    TEMP=/app/temp \
    TMP=/app/temp

# 创建目录并设置权限
RUN mkdir -p /app/temp && chown -R appuser:appgroup /app/temp
```

**为什么不用/tmp**：
- `/tmp` 可能被其他进程或主机清理
- 权限可能不够
- 大量临时文件可能耗尽主机的tmpfs空间

### 3.4 其他安全加固措施

```dockerfile
# ===== 完整安全加固清单 =====

# 1. 使用特定的版本标签，不要用latest
# FROM mcr.microsoft.com/dotnet/aspnet:8.0  ✅
# FROM mcr.microsoft.com/dotnet/aspnet:latest   ❌

# 2. 不要在镜像中存储密钥
# ENV ConnectionString="Server=..."  ❌ 绝对禁止！
# 应该通过环境变量、Docker secrets或K8s ConfigMap注入

# 3. 限制 capabilities（需要docker run或compose配置）
# docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myimage

# 4. 设置只读根文件系统（需要挂载可写卷）
# docker run --read-only --tmpfs /tmp --tmpfs /app/temp myimage

# 5. 禁用不必要的功能
ENV DOTNET_DISABLE_GCMEMORYPRESSURE=0  # 根据实际需求调整
```

---

## 四、生产级docker-compose配置

### 4.1 docker-compose.yml vs docker-compose.prod.yml

永远不要在生产环境使用开发用的compose文件。应该维护两个版本：

```yaml
# docker-compose.yml - 开发环境配置
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile.dev  # 开发专用Dockerfile
    ports:
      - "5000:8080"
      - "5001:8081"  # 调试端口
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:8081;http://+:8080
    volumes:
      - ./src:/app              # 热重载：挂载源代码
      - ~/.nuget/packages:/root/.nuget/packages  # 缓存NuGet包
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=YourStrong@Password123
    ports:
      - "1433:1433"
    volumes:
      - mssql_data:/var/opt/mssql
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P '$$MSSQL_SA_PASSWORD' -Q 'SELECT 1'"]
      interval: 10s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mssql_data:
  redis_data:

networks:
  default:
    name: dev-network
```

### 4.2 docker-compose.prod.yml - 生产环境配置

```yaml
# docker-compose.prod.yml - 生产环境专用配置
# 使用方式: docker compose -f docker-compose.prod.yml up -d
version: '3.8'

services:
  api:
    image: ${REGISTRY}/myapi:${IMAGE_TAG:-latest}
    restart: unless-stopped
    expose:
      - "8080"
    environment:
      # 生产环境变量（敏感值通过.env.prod或secrets注入）
      - ASPNETCORE_ENVIRONMENT=${ASPNETCORE_ENVIRONMENT:-Production}
      - ASPNETCORE_URLS=http://+:8080
      - ConnectionStrings__DefaultConnection=${DB_CONNECTION_STRING}
      - Redis__ConnectionString=${REDIS_CONNECTION_STRING}
      - Serilog__Seq__ServerUrl=${SEQ_SERVER_URL}
      # Application Insights（可选）
      - APPLICATIONINSIGHTS_CONNECTION_STRING=${APPLICATIONINSIGHTS_CONNECTION_STRING}
    # ===== 资源限制（生产必备）=====
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 256M
      replicas: 2  # 多实例保证可用性
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 5
        window: 120s
    # ===== 日志配置 =====
    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "5"
        labels: "service,environment"
        tag: "{{.Name}}"
    # ===== 安全配置 =====
    security_opt:
      - no-new-privileges:true
    read_only: true  # 只读根文件系统
    tmpfs:
      - /tmp:size=100M,mode=1777
      - /app/temp:size=200M,mode=1777
    # ===== 健康检查 =====
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    # ===== 网络隔离 =====
    networks:
      - backend
      - monitoring  # 监控网络
    # ===== 依赖关系 =====
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    dns:
      - 8.8.8.8
      - 8.8.4.4

  # ========== 数据库服务 ==========
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    restart: unless-stopped
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=${MSSQL_SA_PASSWORD}
      - MSSQL_COLLATION=Chinese_PRC_CI_AS
    expose:
      - "1433"
    volumes:
      - mssql_prod_data:/var/opt/mssql
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
    logging:
      driver: json-file
      options:
        max-size: "100m"
        max-file: "3"
    networks:
      - backend
    healthcheck:
      test: >
        CMD-SHELL /opt/mssql-tools18/bin/sqlcmd -S localhost
        -U sa -P "$$MSSQL_SA_PASSWORD"
        -C -Q "SELECT 1" || exit 1
      interval: 15s
      timeout: 10s
      retries: 10
      start_period: 30s

  # ========== Redis缓存 ==========
  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: >
      redis-server
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --requirepass ${REDIS_PASSWORD}
      --appendonly yes
      --appendfsync everysec
      --save 900 1
      --save 300 10
      --save 60 10000
    expose:
      - "6379"
    volumes:
      - redis_prod_data:/data
    deploy:
      resources:
        limits:
          memory: 384M
        reservations:
          memory: 128M
    logging:
      driver: json-file
      options:
        max-size: "20m"
        max-file: "3"
    networks:
      - backend
    sysctls:
      net.core.somaxconn: 1024
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s

  # ========== 反向代理（Nginx）==========
  nginx:
    image: nginx:1.25-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./ssl/certs:/etc/nginx/ssl:ro
      - nginx_logs:/var/log/nginx
    depends_on:
      - api
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 128M
    logging:
      driver: json-file
      options:
        max-size: "20m"
        max-file: "3"
    networks:
      - frontend
      - backend
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 5s
      retries: 3

  # ========== 日志收集（Fluent Bit）==========
  fluentbit:
    image: cr.fluentbit.io/fluent/fluent-bit:2.1.9
    restart: unless-stopped
    volumes:
      - ./fluentbit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - monitoring
    deploy:
      resources:
        limits:
          memory: 128M

# ========== 持久化卷 ==========
volumes:
  mssql_prod_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/mssql
  redis_prod_data:
    driver: local
  nginx_logs:
    driver: local

# ========== 网络定义 ==========
networks:
  frontend:
    name: prod-frontend
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
  backend:
    name: prod-backend
    driver: bridge
    internal: true  # 内部网络，不可从外部访问
    ipam:
      config:
        - subnet: 172.29.0.0/16
  monitoring:
    name: prod-monitoring
    driver: bridge
```

### 4.3 .env.prod 环境变量文件

```bash
# .env.prod - 生产环境变量（不要提交到Git！）
# 此文件应通过安全的密钥管理系统分发

# ===== 应用配置 =====
ASPNETCORE_ENVIRONMENT=Production
IMAGE_TAG=v1.2.0
REGISTRY=myregistry.azurecr.io

# ===== 数据库连接 =====
DB_CONNECTION_STRING=Server=db,1433;Database=MyDb;User Id=sa;Password=${MSSQL_SA_PASSWORD};TrustServerCertificate=True;
MSSQL_SA_PASSWORD=YourSuperSecurePassword!@#2024

# ===== Redis配置 =====
REDIS_CONNECTION_STRING=db,6379,password=${REDIS_PASSWORD},abortConnect=False
REDIS_PASSWORD=AnotherSecurePassword123!

# ===== 监控配置 =====
SEQ_SERVER_URL=https://seq.mycompany.com
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=xxx;IngestionEndpoint=https://xxx.in.applicationinsights.azure.com/

# ===== SSL证书路径（如果使用Let's Encrypt）=====
SSL_CERTIFICATE_PATH=/etc/nginx/ssl/fullchain.pem
SSL_CERTIFICATE_KEY_PATH=/etc/nginx/ssl/privkey.pem
```

---

## 五、容器编排健康检查

### 5.1 三种探针类型详解

在容器编排平台（如Kubernetes）中，健康检查分为三种类型，每种有不同的用途：

```mermaid
graph TB
    subgraph "Liveness Probe 存活探针"
        L1[目的: 应用是否还活着?]
        L2[触发: 应用崩溃/死锁]
        L3[动作: 重启容器]
        L4["检查项: /health/live<br/>轻量级检查"]
    end

    subgraph "Readiness Probe 就绪探针"
        R1[目的: 是否准备好接收流量?]
        R2[触发: 依赖未就绪/过载]
        R3[动作: 从Service移除<br/>不再接收新请求]
        R4["检查项: /health/ready<br/>检查数据库/Redis等依赖"]
    end

    subgraph "Startup Probe 启动探针"
        S1[目的: 应用是否启动完成?]
        S2[触发: 慢启动应用]
        S3[动作: 防止误杀慢启动的应用]
        S4["检查项: /health/startup<br/>或复用live/ready"]
    end

    style L1 fill:#ff6b6b,color:#fff
    style R1 fill:#fcc419,color:#333
    style S1 fill:#51cf66,color:#fff
```

### 5.2 ASP.NET Core健康检查实现

```csharp
// Program.cs - 完整的健康检查配置
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

var builder = WebApplication.CreateBuilder(args);

// ========== 注册健康检查服务 ==========
builder.Services.AddHealthChecks()
    // ----- 存活检查（轻量）-----
    // 只检查应用本身是否响应，不检查外部依赖
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: new[] { "live" })

    // ----- 就绪检查（重量）-----
    // 检查所有关键依赖是否可用
    .AddSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")!,
        name: "database",
        tags: new[] { "ready" },
        timeout: TimeSpan.FromSeconds(5))
    .AddRedis(
        builder.Configuration["Redis:ConnectionString"] ?? "localhost:6379",
        name: "redis-cache",
        tags: new[] { "ready" },
        timeout: TimeSpan.FromSeconds(3))

    // 自定义健康检查：检查磁盘空间
    .AddCheck<DiskSpaceHealthCheck>("disk-space", tags: new[] { "ready" })

    // 自定义健康检查：检查外部API依赖
    .AddCheck<ExternalApiHealthCheck>("external-api", tags: new[] { "ready" });

var app = builder.Build();

// ========== 映射健康检查端点 ==========

// 存活探针端点（供K8s/Docker livenessProbe调用）
app.MapHealthChecks("/health/live", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("live"),
    ResponseWriter = WriteHealthResponse
});

// 就绪探针端点（供K8s/Docker readinessProbe调用）
app.MapHealthChecks("/health/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready"),
    ResponseWriter = WriteHealthResponse
});

// 完整健康检查端点（用于人工查看）
app.MapHealthChecks("/health", new HealthCheckOptions
{
    Predicate = _ => true,
    ResponseWriter = WriteHealthResponse
});

app.Run();

// ========== 响应格式化器 ==========
static async Task WriteHealthResponse(HttpContext context, HealthReport report)
{
    context.Response.ContentType = "application/json";

    var response = new
    {
        status = report.Status.ToString(),
        totalDurationMs = report.TotalDuration.TotalMilliseconds,
        checks = report.Entries.Select(e => new
        {
            name = e.Key,
            status = e.Value.Status.ToString(),
            durationMs = e.Value.Duration.TotalMilliseconds,
            description = e.Value.Description,
            error = e.Value.Exception?.Message,
            data = e.Value.Data
        })
    };

    await context.Response.WriteAsJsonAsync(response);
}

// ========== 自定义健康检查实现 ==========

// 磁盘空间检查
public class DiskSpaceHealthCheck : IHealthCheck
{
    private readonly long _minFreeBytes;

    public DiskSpaceHealthCheck(IConfiguration config)
    {
        // 从配置读取最小剩余空间要求（默认1GB）
        _minFreeBytes = config.GetValue<long>("HealthChecks:DiskMinFreeBytes", 1073741824);
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var driveInfo = new DriveInfo(AppDomain.CurrentDomain.BaseDirectory);
            var freeBytes = driveInfo.AvailableFreeSpace;

            if (freeBytes < _minFreeBytes)
            {
                return HealthCheckResult.Unhealthy(
                    description: $"磁盘空间不足: 剩余{freeBytes / 1024 / 1024}MB, 要求至少{_minFreeBytes / 1024 / 1024}MB",
                    data: new { freeMB = freeBytes / 1024 / 1024, requiredMB = _minFreeBytes / 1024 / 1024 });
            }

            return HealthCheckResult.Healthy(
                description: $"磁盘空间充足: 剩余{freeBytes / 1024 / 1024}MB",
                data: new { freeMB = freeBytes / 1024 / 1024 });
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy(exception: ex);
        }
    }
}

// 外部API依赖检查
public class ExternalApiHealthCheck : IHealthCheck
{
    private readonly HttpClient _httpClient;
    private readonly string _externalApiUrl;
    private readonly ILogger<ExternalApiHealthCheck> _logger;

    public ExternalApiHealthCheck(IHttpClientFactory httpClientFactory, IConfiguration config, ILogger<ExternalApiHealthCheck> logger)
    {
        _httpClient = httpClientFactory.CreateClient("ExternalApi");
        _externalApiUrl = config["ExternalServices:PaymentApi:HealthUrl"] ?? throw new ArgumentNullException(nameof(config));
        _logger = logger;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            using var cts = CancellationTokenSource.CreateLinkedTokenSource(cancellationToken);
            cts.CancelAfter(TimeSpan.FromSeconds(5)); // 5秒超时

            var response = await _httpClient.GetAsync(_externalApiUrl, cts.Token);

            if (response.IsSuccessStatusCode)
            {
                return HealthCheckResult.Healthy($"外部API {_externalApiUrl} 可访问");
            }

            return HealthCheckResult.Degraded(
                description: $"外部API返回状态码: {(int)response.StatusCode}",
                data: new { statusCode = (int)response.StatusCode, url = _externalApiUrl });
        }
        catch (HttpRequestException ex)
        {
            _logger.LogWarning(ex, "外部API健康检查失败: {Url}", _externalApiUrl);
            return HealthCheckResult.Unhealthy(description: "无法连接到外部API", exception: ex);
        }
        catch (TaskCanceledException ex)
        {
            _logger.LogWarning(ex, "外部API健康检查超时: {Url}", _externalApiUrl);
            return HealthCheckResult.Unhealthy(description: "外部API响应超时", exception: ex);
        }
    }
}
```

### 5.3 Kubernetes探针配置示例

```yaml
# k8s-deployment.yaml - K8s部署配置中的探针定义
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapi
  template:
    metadata:
      labels:
        app: myapi
    spec:
      containers:
      - name: myapi
        image: myregistry/myapi:v1.2.0
        ports:
        - containerPort: 8080
        # ===== 存活探针 =====
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30    # 首次检查延迟（让应用有时间启动）
          periodSeconds: 15           # 检查间隔
          timeoutSeconds: 5          # 单次超时
          failureThreshold: 3        # 连续失败次数后重启
          successThreshold: 1
        # ===== 就绪探针 =====
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 10    # 可以比liveness短
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3        # 连续3次失败就从Service摘除
          successThreshold: 1
        # ===== 启动探针（用于慢启动应用）=====
        startupProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30        # 允许最多5分钟（30*10秒）的启动时间
          successThreshold: 1
        # ===== 资源限制 =====
        resources:
          requests:
            cpu: "250m"              # 请求0.25核
            memory: "256Mi"          # 请求256MB内存
          limits:
            cpu: "1000m"             # 最多使用1核
            memory: "512Mi"          # 最多使用512MB
        # ===== 安全上下文 =====
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 10001           # 对应Dockerfile中的appuser UID
          allowPrivilegeEscalation: false
        # ===== 挂载点 =====
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: app-temp
          mountPath: /app/temp
      volumes:
      - name: tmp
        emptyDir: {}
      - name: app-temp
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: myapi-service
spec:
  selector:
    app: myapi
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

---

## 六、资源限制与请求

### 6.1 为什么必须设置资源限制

不设资源限制的后果：

```mermaid
sequenceDiagram
    participant Container as 容器A
    participant Host as 主机
    participant Other as 其他容器

    Note over Container: 内存泄漏或突发流量
    Container->>Host: 占用所有可用内存
    Host->>Other: OOM Killer杀死其他容器
    Note over Other: ❌ 服务中断！

    Note over Container: CPU密集型操作
    Container->>Host: 占用100% CPU
    Note over Host & Other: 所有服务卡顿
```

### 6.2 如何确定合适的资源值

#### 方法1：基准测试

```csharp
// 测试不同负载下的资源占用
public class ResourceBenchmark
{
    private readonly HttpClient _httpClient = new();

    [Benchmark]
    public async Task BaselineLoad()
    {
        // 模拟正常流量
        var tasks = Enumerable.Range(0, 100)
            .Select(_ => _httpClient.GetAsync("https://api.example.com/api/posts"));
        await Task.WhenAll(tasks);
    }

    [Benchmark]
    public async Task HighLoad()
    {
        // 模拟高峰流量
        var tasks = Enumerable.Range(0, 1000)
            .Select(_ => _httpClient.GetAsync("https://api.example.com/api/posts"));
        await Task.WhenAll(tasks);
    }
}
```

#### 方法2：监控生产环境

```bash
# 使用docker stats实时查看资源使用情况
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# 输出示例：
# NAME                CPU %     MEM USAGE / LIMIT     NET I/O           BLOCK I/O
# myapi_api_1         12.35%    185.2MiB / 1GiB      125kB / 156kB     45.6MB / 12.3MB
# myapi_db_1          8.42%     512.4MiB / 2GiB      89kB / 102kB       234MB / 56.7MB
```

#### 方法3：经验值参考

| 应用类型 | CPU Request | CPU Limit | Memory Request | Memory Limit |
|---------|------------|-----------|----------------|--------------|
| Web API（轻量） | 100m | 500m | 128Mi | 256Mi |
| Web API（标准） | 250m | 1000m | 256Mi | 512Mi |
| Web API（重型） | 500m | 2000m | 512Mi | 1Gi |
| 后台Worker | 100m | 500m | 64Mi | 256Mi |
| GraphQL网关 | 500m | 2000m | 512Mi | 1Gi |

> **公式**：Request = 平均使用量的1.5倍；Limit = 峰值的1.2-1.5倍

### 6.3 .NET应用的内存调优

```csharp
// Program.cs - 配置.NET垃圾回收器
var builder = WebApplication.CreateBuilder(args);

// 根据容器内存限制自动调整GC配置
var memoryLimitGb = Environment.GetEnvironmentVariable("MEMORY_LIMIT_GB");
if (!string.IsNullOrEmpty(memoryLimitGb) && double.TryParse(memoryLimitGb, out var limitGb))
{
    // Server GC模式（多核场景性能更好）
    AppContext.SetData("GCServer", true);

    // 设置堆硬限制（留一些给非堆内存）
    var heapLimitMb = (int)(limitGb * 1024 * 0.75); // 使用75%的内存作为堆上限
    Environment.SetEnvironmentVariable("DOTNET_GCHeapHardLimit", (heapLimitMb * 1024 * 1024).ToString());

    // 设置高内存阈值（触发完整GC的堆占比）
    Environment.SetEnvironmentVariable("DOTNET_GCHighMemPercent", "75");

    builder.Logging.LogInformation("GC配置: 堆限制={HeapLimit}MB, 总内存={TotalMemory}GB",
        heapLimitMb, limitGb);
}
```

---

## 七、日志驱动配置

### 7.1 Docker日志驱动选项

| 驱动 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| `json-file` | 开发/小规模生产 | 简单、无需额外组件 | 不自动轮转会占满磁盘 |
| `local` | 小规模生产 | 比json-file更紧凑 | 同上 |
| `fluentd` | 中大规模 | 可转发到多种后端 | 需要额外Fluentd服务 |
| `syslog` | 已有syslog基础设施 | 与现有日志系统集成 | 格式受限 |
| `none` | 有独立日志方案 | 完全不占用Docker日志 | 需自行处理日志收集 |

### 7.2 推荐的json-file配置

```yaml
# docker-compose.prod.yml 中的日志配置
services:
  api:
    logging:
      driver: json-file
      options:
        max-size: "50m"       # 单个日志文件最大50MB
        max-file: "5"         # 最多保留5个文件（共250MB）
        labels: "service,environment,version"  # 添加元数据标签
        tag: "{{.Name}}"      # 日志标签包含容器名
```

### 7.3 Fluentd集成（适合Kubernetes）

```yaml
# docker-compose.prod.yml 中使用Fluentd驱动
services:
  api:
    logging:
      driver: fluentd
      options:
        fluentd-address: "localhost:24224"
        tag: "docker.{{.Name}}"
        fluentd-async-connect: "true"
        fluentd-request-ack: "true"
        fluentd-request-timeout: "30s"
```

配合Fluentd配置文件：

```xml
<!-- fluentd.conf -->
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<filter docker.**>
  @type record_transformer
  <record>
    service_name ${tag_parts[1]}
    hostname ${hostname}
    container_id ${container_id}
    container_name ${container_name}
    image_name ${image_name}
  </record>
</filter>

<match docker.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix docker-logs
  <buffer>
    @type file
    path /var/log/fluentd-buffers/docker
    flush_mode interval
    flush_interval 5s
    chunk_limit_size 2M
    queue_limit_length 8
    retry_max_interval 30s
    retry_forever true
  </buffer>
</match>
```

---

## 八、网络隔离策略

### 8.1 Bridge网络（单机多容器）

```mermaid
graph TB
    subgraph "Docker Bridge Network"
        Internet((🌐 Internet)) -->|Port 80/443| Nginx[Nginx反向代理]
        Nginx -->|内部网络| API[API容器 :8080]
        API -->|内部网络| DB[(SQL Server :1433)]
        API -->|内部网络| Redis[(Redis :6379)]
        DB -.->|不可达| Internet
        Redis -.->|不可达| Internet
        API -.->|不可达| Internet
    end

    style Nginx fill:#74c0fc,color:#333
    style DB fill:#fcc419,color:#333
    style Redis fill:#fcc419,color:#333
```

```yaml
# docker-compose.prod.yml 中的网络隔离配置
networks:
  frontend:
    # 外部可访问的网络
    driver: bridge
    name: prod-public
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16

  backend:
    # 内部网络，禁止外部直接访问
    driver: bridge
    internal: true  # 关键：设置为内部网络
    name: prod-private
    ipam:
      driver: default
      config:
        - subnet: 172.29.0.0/16

  monitoring:
    # 监控专用网络
    driver: bridge
    name: prod-monitoring

services:
  nginx:
    networks:
      - frontend   # 对外暴露
      - backend    # 转发到后端
    ports:
      - "80:80"
      - "443:443"

  api:
    networks:
      - backend    # 只在内网通信
      - monitoring # 上报监控数据
    # 不暴露端口到主机！
    # expose只在容器间可见
    expose:
      - "8080"

  db:
    networks:
      - backend
    # 数据库绝对不应该暴露到外部
    expose:
      - "1433"
```

### 8.2 Overlay网络（多机集群/Swarm）

当需要在多台服务器上运行容器时，使用Overlay网络：

```bash
# 创建overlay网络（需要Swarm或Kubernetes）
docker network create -d overlay \
    --subnet=10.0.0.0/24 \
    --gateway=10.0.0.1 \
    --attachable \
    prod-overlay-network
```

### 8.3 网络安全最佳实践

```markdown
## 网络安全清单

### 必做项
- [ ] 数据库和Redis只加入internal网络
- [ ] API服务不绑定到0.0.0.0（使用expose而非ports）
- [ ] 通过Nginx统一入口，不在多个服务上重复暴露端口
- [ ] 使用防火墙规则限制Docker主机端口访问

### 进阶项
- [ ] 实施网络策略（Network Policy）控制Pod间通信
- [ ] 启用TLS加密容器间通信
- [ ] 定期审查网络配置，移除不必要的端口映射
- [ ] 使用服务网格（Istio/Linkerd）实现零信任网络
```

---

## 九、Secrets敏感信息管理

### 9.1 绝对不要做的事情

```dockerfile
# ❌ 致命错误：在Dockerfile中硬编码密钥
ENV DB_PASSWORD="Admin@123"
ENV API_KEY="sk-abc123456789"

# ❌ 严重错误：在镜像中包含配置文件
COPY appsettings.Production.json .

# ❌ 错误：在构建参数中传递密钥
ARG CONNECTION_STRING
ENV ConnectionString=$CONNECTION_STRING  # ARG会留在镜像历史中！
```

### 9.2 Docker Secrets（Swarm模式）

```yaml
# docker-compose.prod.yml - 使用Docker Secrets
services:
  api:
    image: myregistry/myapi:v1.2.0
    secrets:
      - db_password
      - redis_password
      - jwt_secret_key
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyDb;User Id=sa;Password={{`/run/secrets/db_password`}};

secrets:
  db_password:
    external: true
  redis_password:
    external: true
  jwt_secret_key:
    external: true
```

```bash
# 创建secret（在Swarm模式下）
echo "MySuperSecurePassword123!" | docker secret create db_password -
echo "AnotherSecurePass" | docker secret create redis_password -

# 查看secret列表
docker secret ls

# 删除secret（轮换时）
docker secret rm db_password
echo "NewSecurePassword2024" | docker secret create db_password -
```

### 9.3 环境变量文件（.env）

```bash
# .env.production - 敏感环境变量（绝不能提交到Git！）
# 添加到.gitignore: .env.production

# 数据库
DB_CONNECTION_STRING=Server=prod-db.example.com,1433;Database=ProdDb;...
MSSQL_SA_PASSWORD=ComplexPassword!@#$2024

# Redis
REDIS_PASSWORD=RedisSecurePass123!

# JWT
JWT_SECRET_KEY=your-very-long-and-random-secret-key-minimum-32-characters
JWT_ISSUER=https://api.mycompany.com
JWT_AUDIENCE=https://mycompany.com

# 第三方服务
STRIPE_API_KEY=sk_live_xxxxxxxxxxxxxx
SENDGRID_API_KEY SG.xxxxxxxxxxxxxx

# 使用方式
# docker compose --env-file .env.production -f docker-compose.prod.yml up -d
```

### 9.4 Kubernetes Secret管理

```yaml
# k8s-secrets.yaml - Kubernetes Secret定义
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  # Base64编码的值（或者使用stringData自动编码）
  connection-string: "Server=db-service,1433;Database=MyDb;User Id=sa;Password=SecurePass123;"
  redis-password: "RedisSecurePassword!"
  jwt-secret-key: "your-super-secret-jwt-key-at-least-32-chars"
---
apiVersion: v1
kind: Secret
metadata:
  name: tls-cert
type: kubernetes.io/tls
stringData:
  tls.crt: |
    -----BEGIN CERTIFICATE-----
    MIIExxxxxx...你的证书内容...xxxxxx
    -----END CERTIFICATE-----
  tls.key: |
    -----BEGIN RSA PRIVATE KEY-----
    MIIExxxxxx...你的私钥内容...xxxxxx
    -----END RSA PRIVATE KEY-----
```

在Deployment中引用：

```yaml
spec:
  containers:
  - name: myapi
    envFrom:
    - secretRef:
        name: app-secrets
    volumeMounts:
    - name: tls-volume
      mountPath: /etc/tls
      readOnly: true
  volumes:
  - name: tls-volume
    secret:
      secretName: tls-cert
```

### 9.5 Azure Key Vault集成（推荐企业级方案）

```csharp
// Program.cs - 使用Azure Key Vault作为配置源
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    // 使用Managed Identity访问Key Vault
    builder.Configuration.AddAzureKeyVault(
        new Uri($"https://{builder.Configuration["KeyVault:VaultName"]}.vault.azure.net/"),
        new DefaultAzureCredential());
}

// 现在可以直接像读取普通配置一样读取密钥
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
// 这个值实际上来自Key Vault，但代码完全无感知
```

---

## 十、完整生产级模板

### 10.1 最终版Dockerfile

```dockerfile
# ============================================================
# ASP.NET Core 8 生产级终极Dockerfile
# 版本: v2.0 - 2024年最新最佳实践
# 特征: 安全加固 + 多阶段构建 + 最小化镜像 + 完整健康检查
# ============================================================

# ==================== 阶段1: 还原依赖 ====================
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS restore
WORKDIR /src

# 只复制项目文件以最大化缓存利用率
COPY ["MyApi/MyApi.csproj", "MyApi/"]
COPY ["NuGet.Config", "./"]

# 还原依赖（此层会被缓存直到csproj改变）
RUN dotnet restore "MyApi/MyApi.csproj" \
    --configfile NuGet.Config

# ==================== 阶段2: 编译与测试 ====================
FROM restore AS build
ARG BUILD_CONFIGURATION=Release

# 复制完整的解决方案文件
COPY . .

# 编译
RUN dotnet build "MyApi/MyApi.csproj" \
    -c $BUILD_CONFIGURATION \
    --no-restore \
    -o /app/build

# 测试
RUN dotnet test "MyApi.Tests/MyApi.Tests.csproj" \
    -c $BUILD_CONFIGURATION \
    --no-build \
    --verbosity normal \
    --results-directory /test-results \
    --logger "trx;LogFileResults=/test-results/results.trx"

# ==================== 阶段3: 发布 ====================
FROM build AS publish
RUN dotnet publish "MyApi/MyApi.csproj" \
    -c $BUILD_CONFIGURATION \
    -o /app/publish \
    --self-contained false \
    -p:UseAppHost=false

# ==================== 阶段4: 最终运行镜像 ====================
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final

# 安装运行时依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        tini \
        ca-certificates \
        icu-libs \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 创建非特权用户
RUN groupadd --gid 10001 appgroup && \
    useradd --uid 10001 --gid appgroup --create-home --shell /usr/sbin/nologin appuser

# 创建必要目录
RUN mkdir -p /app/{logs,temp} && \
    chown -R appuser:appgroup /app

# 环境变量
ENV ASPNETCORE_URLS=http://+:8080 \
    ASPNETCORE_ENVIRONMENT=Production \
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false \
    DOTNET_RUNNING_IN_CONTAINER=true \
    TZ=Asia/Shanghai \
    TMPDIR=/app/temp \
    TEMP=/app/temp \
    # GC优化
    DOTNET_GCServer=1 \
    DOTNET_TieredCompilation=1 \
    # 日志
    Logging__Console__FormatterName=json

# 切换到非root用户
USER appuser
WORKDIR /app

# 复制应用
COPY --from=publish --chown=appuser:appgroup /app/publish .

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl --fail http://localhost:8080/health/live || exit 1

EXPOSE 8080

# 使用tini作为init进程
ENTRYPOINT ["tini", "--"]
CMD ["dotnet", "MyApi.dll"]
```

### 10.2 构建和部署脚本

```powershell
# build-and-push.ps1 - 构建并推送生产镜像
param(
    [Parameter(Mandatory)]
    [string]$Registry,          # 例如: myregistry.azurecr.io

    [Parameter(Mandatory)]
    [string]$ImageName,         # 例如: myapi

    [Parameter()]
    [string]$Tag = "latest",

    [switch]$Push               # 是否推送到仓库
)

$ErrorActionPreference = "Stop"
$fullImage = "$Registry/$ImageName:$Tag"

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "  构建 Docker 镜像" -ForegroundColor Cyan
Write-Host "  镜像: $fullImage" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# Step 1: 构建镜像（使用BuildKit获得更好的缓存和并行构建）
Write-Host "`n[1/4] 开始构建镜像..." -ForegroundColor Yellow
DOCKER_BUILDKIT=1 docker build `
    --target final `
    -t $fullImage `
    --build-arg BUILD_CONFIGURATION=Release `
    --cache-from=$Registry/$ImageName:cache `
    --cache-to=type=registry,ref=$Registry/$ImageName:cache,mode=max `
    -f Dockerfile `
    .

if ($LASTEXITCODE -ne 0) {
    Write-Host "❌ 构建失败！" -ForegroundColor Red
    exit 1
}

# Step 2: 扫描安全漏洞
Write-Host "`n[2/4] 扫描镜像安全漏洞..." -ForegroundColor Yellow
docker scan $fullImage 2>$null
# 或者使用Trivy: trivy image --severity HIGH,CRITICAL $fullImage

# Step 3: 查看镜像信息
Write-Host "`n[3/4] 镜像信息:" -ForegroundColor Green
docker images $fullImage --format "  大小: {{.Size}}, 创建时间: {{.CreatedAt}}"

# Step 4: 推送（如果指定）
if ($Push) {
    Write-Host "`n[4/4] 推送到注册表..." -ForegroundColor Yellow
    docker push $fullImage
    Write-Host "✅ 镜像推送成功！" -ForegroundColor Green
} else {
    Write-Host "`n[4/4] 跳过推送（使用 -Push 参数推送）" -ForegroundColor Gray
}

Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "  完成！镜像: $fullImage" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan
```

### 10.3 安全扫描脚本

```bash
#!/bin/bash
# scan-images.sh - Docker镜像安全扫描

set -e

REGISTRY="${1:-myregistry.azurecr.io}"
IMAGE_NAME="${2:-myapi}"
TAG="${3:-latest}"

FULL_IMAGE="$REGISTRY/$IMAGE_NAME:$TAG"

echo "=========================================="
echo "  Docker 镜像安全扫描"
echo "  镜像: $FULL_IMAGE"
echo "=========================================="

# 方法1: 使用Trivy（推荐）
if command -v trivy &> /dev/null; then
    echo ""
    echo "[Trivy] 扫描漏洞..."
    trivy image \
        --severity HIGH,CRITICAL \
        --exit-code 1 \
        --no-progress \
        --format table \
        "$FULL_IMAGE"
else
    echo "⚠️  Trivy未安装，使用docker scan替代..."
    docker scan "$FULL_IMAGE"
fi

# 方法2: 检查镜像是否符合最佳实践
echo ""
echo "[Docker Lint] 检查Dockerfile最佳实践..."
if command -v hadolint &> /dev/null; then
    hadolint Dockerfile
else
    echo "ℹ️  hadolint未安装，跳过"
fi

# 方法3: 检查镜像大小
echo ""
echo "[Size Check] 镜像大小:"
docker images "$FULL_IMAGE" --format "  {{.Repository}}:{{.Tag}} - {{.Size}}"

SIZE=$(docker images "$FULL_IMAGE" --format "{{.Size}}")
SIZE_MB=$(echo "$SIZE" | grep -oE '[0-9]+' | head -1)

if [ "$SIZE_MB" -gt 300 ]; then
    echo "⚠️  警告: 镜像超过300MB，考虑进一步优化"
elif [ "$SIZE_MB" -gt 200 ]; then
    echo "ℹ️  提示: 镜像在200-300MB之间，可以接受"
else
    echo "✅ 镜像大小良好 (<200MB)"
fi

echo ""
echo "=========================================="
echo "  扫描完成"
echo "=========================================="
```

---

## 总结

生产环境的Docker配置与开发环境有着天壤之别。通过本文学到的知识，你应该能够掌握：

✅ **理解核心差异**：开发追求便捷快速，生产追求安全稳定  
✅ **掌握多阶段构建**：将镜像体积从GB级别降到百MB级别  
✅ **实施安全加固**：非root用户、只读文件系统、capabilities限制  
✅ **编写生产Compose**：资源限制、健康检查、日志配置、网络隔离  
✅ **配置三种探针**：Liveness/Readiness/Startup保障应用可靠性  
✅ **安全管理密钥**：Docker Secrets/K8s Secrets/Azure Key Vault  

**关键要点回顾**：

| 最佳实践 | 重要性 | 实施难度 |
|---------|--------|---------|
| 多阶段构建 | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 非root用户运行 | ⭐⭐⭐⭐⭐ | ⭐ |
| HEALTHCHECK | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 资源限制 | ⭐⭐⭐⭐⭐ | ⭐ |
| .dockerignore | ⭐⭐⭐⭐ | ⭐ |
| 密钥外部化管理 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 网络隔离 | ⭐⭐⭐⭐ | ⭐⭐ |
| 结构化日志 | ⭐⭐⭐⭐ | ⭐⭐ |

**下一步学习**：
- [[03-GitHub Actions CI-CD流水线]] - 自动化构建和部署Docker镜像
- [[06-健康检查与优雅关闭]] - 更深入的健康检查机制
- [[08-集中式日志解决方案]] - 生产级日志收集和分析
- [[02-微服务架构/Kubernetes实战部署]] - 容器编排高级主题

---

> **相关文章**：
> - [[02-微服务架构/容器化Docker基础]] - Docker基础知识
> - [[07-Application Insights监控]] - 应用性能监控
> - [[04-安全加固/05-HTTPS与安全头部配置]] - 安全加固细节
> - [[05-多环境配置管理]] - 环境配置最佳实践
