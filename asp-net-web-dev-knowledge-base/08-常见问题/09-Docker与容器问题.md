# ASP.NET Docker 与容器问题解答

> **适用场景**：使用Docker容器化ASP.NET应用时遇到的构建、运行和网络问题
>
> **目标读者**：需要进行容器化部署、Docker Compose编排和Kubernetes管理的开发者

---

## 问题1：Docker 镜像构建失败

### 常见错误及解决

```dockerfile
# ✅ 推荐的多阶段构建 Dockerfile
# 阶段1：构建
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# 先复制csproj和NuGet缓存以利用Docker层缓存
COPY ["MyApp.csproj", "./"]
RUN dotnet restore "MyApp.csproj"

# 复制源代码
COPY . .

# 发布
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

# 阶段2：运行
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

# 安装必要的运行时依赖（如curl用于健康检查）
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 80
EXPOSE 443

COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

#### 常见构建错误修复

```bash
# 错误：无法恢复包
# 解决：使用国内镜像源或本地缓存
ARG NUGET_PACKAGE_SOURCE=https://nuget.cdn.azure.cn/v3/index.json

# 或在Dockerfile中配置
RUN dotnet restore \
    --configfile NuGet.Config \
    /p:NuGetPackageSource=https://nuget.cdn.azure.cn/v3/index.json

# 错误：SDK版本不匹配
# 解决：确保Dockerfile中的SDK版本与.csproj的TargetFramework一致
FROM mcr.microsoft.com/dotnet/sdk:8.0  # 对应 net8.0

# 错误：层缓存失效导致每次全量重建
# 解决：合理组织COPY顺序，将变化频繁的文件后复制
```

---

## 问题2：容器启动后立即退出

### Exit Code 快速参考

| Code | 含义 | 排查方向 |
|------|------|---------|
| 0 | 正常退出 | 程序正常结束但应该是常驻进程 |
| 1 | 应用错误 | 查看日志 `docker logs` |
| 137 | OOM Kill | 内存不足，增加 memory limit |
| 139 | Segfault | 本地库不兼容，检查平台 |

### 调试方法

```bash
# 交互式调试
docker run -it --rm --entrypoint sh myimage

# 在容器内手动启动应用
dotnet MyApp.dll --urls http://0.0.0.0:80

# 查看完整日志
docker logs mycontainer --tail 100 -f

# 进入运行中的容器
docker exec -it mycontainer sh
```

---

## 问题3：容器间网络不通

### Docker Compose 网络配置

```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    ports:
      - "8080:80"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    environment:
      - ConnectionStrings__DefaultConnection=Server=db;Database=mydb;User Id=sa;Password=Your@Strong!Password123;TrustServerCertificate=True
      - Redis__Configuration=localhost:6379
    networks:
      - app-network

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=Your@Strong!Password123
    healthcheck:
      test: ["CMD-SHELL", "/opt/mssql-tools18/bin/sqlcmd", "-S", "localhost", "-U", "sa", "-P", "Your@Strong!Password123", "-Q", "SELECT 1"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - mssql-data:/var/opt/mssql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  mssql-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

### DNS 解析测试

```bash
# 测试容器间连通性
docker exec web ping db
docker exec web nslookup db
docker exec web curl -v http://db:1433

# 如果DNS有问题，可以使用extra_hosts作为临时方案
extra_hosts:
  - "host.docker.internal:host-gateway"
```

---

## 问题4：数据卷挂载与权限

### Windows 容器权限问题

```yaml
# 方案A：使用命名卷自动处理权限
volumes:
  app-data:

# 方案B：Linux容器指定用户
services:
  web:
    user: "1000:1000"  # UID:GID
    volumes:
      - ./data:/app/data

# 方案C：初始化脚本设置权限
entrypoint: ["/init.sh"]
# init.sh:
#!/bin/sh
chown -R appuser:appgroup /app/data
exec dotnet MyApp.dll
```

### 数据持久化最佳实践

```yaml
services:
  # 开发环境：绑定挂载（方便查看文件）
  web-dev:
    volumes:
      - ./wwwroot:/app/wwwroot
      - ./logs:/app/logs
      - ./data:/app/data

  # 生产环境：命名卷（性能更好）
  web-prod:
    volumes:
      - wwwroot-data:/app/wwwroot
      - logs-data:/app/logs
      - data-volume:/app/data
```

---

## 问题5：Kubernetes Pod 反复重启 (CrashLoopBackOff)

### 排查步骤

```bash
# 1. 查看 Pod 事件
kubectl describe pod <pod-name>

# 2. 查看 Pod 日志
kubectl logs <pod-name> --previous  # 上一次崩溃的日志

# 3. 进入容器调试
kubectl exec -it <pod-name> -- /bin/sh

# 4. 检查资源限制
kubectl top pod <pod-name>

# 常见原因：
# - 应用程序异常退出 → 检查日志
# - Liveness Probe 失败 → 检查健康检查端点
# - OOM Kill → 增加 resources.limits.memory
# - ConfigMap/Secret 缺失 → 检查 volume mounts
```

### 健康检查配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /health/live
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /health/startup
        port: 80
      failureThreshold: 30
      periodSeconds: 10
```

---

## 总结

| 场景 | 关键命令/配置 | 注意事项 |
|------|---------------|---------|
| 构建 | 多阶段Dockerfile | 层缓存优化 |
| 调试 | docker exec, docker logs | Exit code含义 |
| 网络 | docker-compose network | 服务名作为DNS名 |
| 卷挂载 | permissions, named vs bind | 生产用named卷 |
| K8s | liveness/readiness probes | 资源限制防止OOM |

**核心原则**：
1. **多阶段构建**减小镜像体积
2. **健康检查**确保服务可用性
3. **日志输出到stdout/stderr**便于收集
4. **非root用户运行**提升安全性
5. **资源限制**防止单Pod影响节点

---

> **相关知识库链接**
> - [05-部署相关问题](./05-部署相关问题.md)
> - [07-性能问题排查](./07-性能问题排查.md)
