# dotnet new 模板大全

`dotnet new` 是 .NET CLI 中最常用的命令之一，它允许开发者通过预定义的模板快速创建项目。本文将全面介绍 `dotnet new` 的所有模板、参数配置、自定义模板创建以及最佳实践。

## 一、模板概览与基础用法

### 1.1 基本命令语法

```bash
# 列出所有可用模板
dotnet new --list

# 使用指定模板创建项目
dotnet new <template> [options]

# 获取特定模板的帮助信息
dotnet new <template> -h
```

### 1.2 常用全局参数

| 参数 | 缩写 | 说明 | 示例 |
|------|------|------|------|
| `--name` | `-n` | 项目名称 | `-n MyProject` |
| `--output` | `-o` | 输出目录 | `-o ./src` |
| `--framework` | `--framework` | 目标框架 | `--framework net8.0` |
| `--force` | `--force` | 强制覆盖已有文件 | `--force` |
| `--no-restore` | `--no-restore` | 不自动还原依赖 | `--no-restore` |
| `--dry-run` | `--dry-run` | 预览但不实际创建 | `--dry-run` |

---

## 二、Web 应用模板（12个）

### 2.1 核心 Web 模板

#### web — ASP.NET Core Web App (Model-View-Controller)

```bash
# 创建 MVC 应用
dotnet new mvc -n MyMvcApp

# 创建带认证的 MVC 应用
dotnet new mvc -n MyAuthApp --auth Individual

# 指定框架版本
dotnet new mvc -n MyApp --framework net8.0
```

**适用场景**：传统服务器端渲染应用，需要 SEO 友好的页面。

#### webapi — ASP.NET Core Web API

```bash
# 创建 Web API 项目
dotnet new webapi -n MyApi

# 创建带 OpenAPI/Swagger 的 API
dotnet new webapi -n MyApi --openapi true

# 启用控制器方式
dotnet new webapi -n MyApi -use-controllers true
```

**适用场景**：RESTful API 服务，微服务后端，前后端分离架构。

#### blazorserver — Blazor Server App

```bash
# 创建 Blazor Server 应用
dotnet new blazorserver -n MyBlazorServer

# 创建带认证的应用
dotnet new blazorserver -n MyBlazorApp --auth Individual
```

**适用场景**：企业内部应用，实时协作工具，低延迟要求的交互式 UI。

#### blazorwasm — Blazor WebAssembly App

```bash
# 创建 Blazor WASM 托管模式
dotnet new blazorwasm -n MyBlazorWasm --hosted

# 创建 PWA 模式的 Blazor WASM
dotnet new blazorwasm -n MyBlazorPWA -pwa

# 独立部署模式
dotnet new blazorwasm -n MyStandalone
```

**适用场景**：SPA 单页应用，离线优先应用，需要客户端渲染的场景。

### 2.2 页面与组件模板

#### razorpages — ASP.NET Core Web App (Razor Pages)

```bash
# 创建 Razor Pages 应用
dotnet new razorpages -n MyRazorPages

# 带认证功能
dotnet new razorpages -n SecurePages --auth Individual
```

**适用场景**：简单页面驱动的 Web 应用，表单密集型应用。

#### razorcomponent — Razor Component

```bash
# 创建独立的 Razor 组件
dotnet new razorcomponent -n MyComponent -o ./Components
```

**适用场景**：可复用的 UI 组件库，共享组件开发。

### 2.3 服务通信模板

#### grpc — gRPC Service

```bash
# 创建 gRPC 服务
dotnet new grpc -n MyGrpcService

# 创建 gRPC 客户端
dotnet new grpc -n MyGrpcClient
```

**适用场景**：高性能服务间通信，流式数据处理，移动端后端服务。

#### worker — Worker Service

```bash
# 创建后台工作服务
dotnet new worker -n MyWorkerService
```

**适用场景**：Windows 服务，Linux 守护进程，定时任务处理。

### 2.4 其他 Web 相关模板

| 模板名称 | 说明 | 快捷命令 |
|----------|------|----------|
| `webapp` | ASP.NET Core Web App with minimal APIs | `dotnet new webapp` |
| `angular` | Angular + ASP.NET Core SPA | `dotnet new angular` |
| `react` | React + ASP.NET Core SPA | `dotnet new react` |
| `vue` | Vue.js + ASP.NET Core SPA | `dotnet new vue` |

---

## 三、类库模板（8个）

### 3.1 标准类库

#### classlib — Class Library

```bash
# 创建标准类库
dotnet new classlib -n MyLibrary

# 创建多目标框架类库
dotnet new classlib -n MultiTargetLib --tfm netstandard2.0;net8.0
```

**最佳实践**：
- 类库名称使用清晰的前缀或命名空间
- 启用可空引用类型：`<Nullable>enable</Nullable>`
- 添加 XML 文档注释

### 3.2 测试项目模板

#### xunit — xUnit Test Project

```bash
# 创建 xUnit 测试项目
dotnet new xunit -n MyProject.Tests

# 关联被测项目
cd MyProject.Tests
dotnet add reference ../MyProject/MyProject.csproj
```

#### nunit — NUnit Test Project

```bash
# 创建 NUnit 测试项目
dotnet new nunit -n MyNunitTests
```

#### mstest — MSTest Test Project

```bash
# 创建 MSTest 测试项目
dotnet new mstest -n MyMSTests
```

### 3.3 测试对比

| 特性 | xUnit | NUnit | MSTest |
|------|-------|-------|--------|
| 社区活跃度 | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| 断言风格 | Fluent | Classic + Fluent | Attribute-based |
| 数据驱动测试 | InlineData | TestCase | DataRow |
| 并行执行 | 默认支持 | 需配置 | 需配置 |
| 推荐指数 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

### 3.4 UI 类库模板

#### winforms — Windows Forms App

```bash
# 创建 WinForms 应用
dotnet new winforms -n MyWinFormsApp
```

#### wpf — WPF Application

```bash
# 创建 WPF 应用
dotnet new wpf -n MyWpfApp

# 创建 WPF 类库
dotnet new wpflib -n MyWpfLibrary
```

#### wpfcustomcontrollib — WPF Custom Control Library

```bash
# 创建自定义控件库
dotnet new wpfcustomcontrollib -n MyControls
```

---

## 四、配置文件模板（10个）

### 4.1 版本控制与编辑器配置

#### gitignore — Git Ignore File

```bash
# 创建 .gitignore 文件
dotnet new gitignore

# 支持的语言选项
dotnet new gitignore -lang VisualStudio
dotnet new gitignore -lang CSharp
```

**生成的忽略规则包括**：

```gitignore
## .NET
bin/
obj/
*.user
*.suo
*.cache
*.dll
*.exe

## Visual Studio
.vs/
*.vsconfig
ReSharper*/

## Build results
[Bb]in/
[Oo]bj/

## NuGet
packages/
*.nupkg
project.lock.json
```

#### editorconfig — EditorConfig File

```bash
# 创建 EditorConfig
dotnet new editorconfig
```

**生成的配置示例**：

```ini
# EditorConfig is awesome: https://EditorConfig.org

root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.cs]
indent_size = 4

[*.{json,yml}]
indent_size = 2

[Makefile]
indent_style = tab
```

### 4.2 解决方案与项目管理

#### sln — Solution File

```bash
# 创建空解决方案
dotnet new sln -n MySolution

# 向解决方案添加项目
dotnet sln add src/MyProject/MyProject.csproj
dotnet sln add tests/MyProject.Tests/MyProject.Tests.csproj

# 从文件夹结构创建解决方案
dotnet new sln -n MySolution
dotnet sln add **/*.csproj --in-folder
```

#### globaljson — global.json File

```bash
# 创建 global.json 锁定 SDK 版本
dotnet new globaljson --sdk-version 8.0.400

# 指定允许的版本范围
dotnet new globaljson --sdk-version 8.0.* --roll-forward feature
```

**global.json 配置详解**：

```json
{
  "sdk": {
    "version": "8.0.400",
    "rollForward": "feature",
    "allowPrerelease": false
  }
}
```

**rollForward 选项说明**：

| 值 | 行为 |
|----|------|
| `patch` | 仅补丁版本升级 |
| `minor` | 升级到匹配的次版本 |
| `major` | 升级到匹配的主版本 |
| `latestPatch` | 安装最新的补丁版本 |
| `latestMinor` | 安装最新的次版本 |
| `latestMajor` | 安装最新的主版本 |
| `disable` | 不自动升级 |

### 4.3 包管理配置

#### nugetconfig — NuGet Config

```bash
# 创建 NuGet 配置文件
dotnet new nugetconfig

# 配置私有源
dotnet new nugetconfig --source https://nuget.example.com/v3/index.json
```

**典型配置示例**：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="private" value="https://nuget.example.com/v3/index.json" />
  </packageSources>
  <packageSourceMapping>
    <packageSource key="nuget.org">
      <package pattern="*" />
    </packageSource>
    <packageSource key="private">
      <package pattern="Company.*" />
    </packageSource>
  </packageSourceMapping>
</configuration>
```

### 4.4 其他配置模板

| 模板名称 | 说明 | 用途 |
|----------|------|------|
| `protocol` | Protocol Buffer Definition | 定义 gRPC 服务协议 |
| `dockerfile` | Dockerfile | 容器化部署配置 |
| `directory.build.props` | MSBuild 属性文件 | 统一项目构建配置 |
| `props` | MSBuild Props 文件 | 共享构建属性 |
| `targets` | MSBuild Targets 文件 | 自定义构建逻辑 |

---

## 五、Azure 云服务模板（8个）

### 5.1 Azure Functions

#### azure-function — Azure Functions

```bash
# 创建 Azure Functions 项目
dotnet new azure-function -n MyFunctionApp

# 选择运行时模型
dotnet new azure-function -n IsolatedFunc --worker-runtime dotnet-isolated
dotnet new azure-function -n InProcessFunc --worker-runtime dotnet

# 选择触发器类型
dotnet new azure-function -n HttpFunc --functions-v4
```

**支持的触发器类型**：

| 触发器类型 | 模板标识符 | 说明 |
|------------|------------|------|
| HTTP 触发器 | `http` | REST API 端点 |
| 定时触发器 | `timer` | Cron 定时任务 |
| 队列触发器 | `queue` | Azure Queue Storage |
| Blob 触发器 | `blob` | 存储变更检测 |
| Service Bus | `servicebus` | 消息队列处理 |
| Event Grid | `eventgrid` | 事件驱动架构 |
| Event Hubs | `eventhubs` | 大规模事件流 |
| Cosmos DB | `cosmosdb` | 数据库变更监听 |
| SignalR | `signalr` | 实时消息推送 |
| RabbitMQ | `rabbitmq` | AMQP 消息处理 |

### 5.2 其他 Azure 模板

| 模板名称 | 说明 | 典型用途 |
|----------|------|----------|
| `worker-grpc` | gRPC Worker Service | Azure Container Apps |
| `mvc-auth` | MVC with Authentication | Azure AD 集成 |
| `webapp-auth` | Web App with Auth | B2C 身份验证 |
| `razor-auth` | Razor Pages with Auth | 企业内网应用 |
| `blazorserver-auth` | Blazor Server with Auth | 安全仪表板 |
| `blazorwasm-auth` | Blazor WASM with Auth | SPA 认证 |
| `webapi-auth` | Web API with Auth | JWT Token 服务 |

---

## 六、测试相关模板（6个）

### 6.1 核心测试模板

#### razorclasslib — Razor Class Library

```bash
# 创建 Razor 类库（用于组件测试）
dotnet new razorclasslib -n MyComponentLib

# 支持 RCL 的支持文件
dotnet new razorclasslib -n ThemeLib --support-pages-and-views
```

**适用场景**：
- 可复用 Razor 组件库
- Blazor 组件包发布
- 共享视图/页面

### 6.2 测试项目选择指南

```
┌─────────────────────────────────────────────────────────┐
│                  测试框架选择决策树                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   是否需要数据驱动测试？                                   │
│        │                                                │
│   ┌────┴────┐                                           │
│   │         │                                           │
│  是        否                                            │
│   │         │                                           │
│   ▼         ▼                                           │
│ NUnit     xUnit                                        │
│ (TestCase) (InlineData)                                 │
│                                                         │
│   是否在企业环境？                                        │
│        │                                                │
│   ┌────┴────┐                                           │
│   │         │                                           │
│  是        否                                            │
│   │         │                                           │
│   ▼         ▼                                           │
│ MSTest    xUnit/NUnit                                  │
│ (集成好)   (社区活跃)                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 七、其他实用模板（6个）

### 7.1 控制台应用

#### console — Console Application

```bash
# 创建控制台应用
dotnet new console -n MyApp

# 创建顶级语句风格的控制台应用（.NET 6+）
dotnet new console -n ModernApp --use-program-main

# 创建带 Working Directory 的控制台应用
dotnet new console -n ToolApp
```

**控制台应用进阶配置**：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <!-- 全局隐式 using -->
    <ImplicitUsings>enable</ImplicitUsings>
    <!-- 可空引用类型 -->
    <Nullable>enable</Nullable>
    <!-- 单文件发布 -->
    <PublishSingleFile>true</PublishSingleFile>
    <!-- 自包含部署 -->
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
  </PropertyGroup>
</Project>
```

### 7.2 协议与通信

#### protobuf — Protocol Buffer Definition

```bash
# 创建 proto 文件
dotnet new protobuf -n myservice -o ./Protos
```

**生成的 proto 文件示例**：

```protobuf
syntax = "proto3";

option csharp_namespace = "MyService";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

### 7.3 其他模板汇总

| 模板名称 | 说明 | 适用场景 |
|----------|------|----------|
| `solution` | 解决方案文件 | 多项目管理 |
| `grpc` | gRPC 服务/客户端 | 高性能 RPC |
| `webconfig` | Web 配置文件 | IIS 部署 |
| `rulefile` | Analyzer Rule Set | 代码质量规则 |
| `msbuild-task` | MSBuild 任务 | 构建自动化 |

---

## 八、自定义模板开发

### 8.1 创建自定义模板

#### 步骤一：准备模板内容

```bash
# 创建模板目录结构
mkdir my-custom-template
cd my-custom-template

# 创建模板内容
mkdir content
echo 'namespace {{namespace}}' > content/Program.cs
echo '{' >> content/Program.cs
echo '    Console.WriteLine("Hello from custom template!");' >> content/Program.cs
echo '}' >> content/Program.cs
```

#### 步骤二：创建 template.json

```json
{
  "$schema": "http://json.schemastore.org/template",
  "author": "Your Name",
  "classifications": ["Common", "Console"],
  "identity": "My.Custom.Template",
  "name": "My Custom Template",
  "shortName": "mytemplate",
  "tags": {
    "language": "C#",
    "type": "project"
  },
  "sourceName": "template-name-placeholder",
  "preferNameDirectory": true,
  "symbols": {
    "namespace": {
      "type": "parameter",
      "datatype": "string",
      "defaultValue": "MyNamespace",
      "replaces": "{{namespace}}"
    },
    "enableLogging": {
      "type": "parameter",
      "datatype": "bool",
      "defaultValue": "false"
    }
  }
}
```

#### 步骤三：安装和测试模板

```bash
# 从本地路径安装
dotnet new install ./my-custom-template

# 从 NuGet 包安装
dotnet new install MyCompany.Templates::1.0.0

# 使用自定义模板
dotnet new mytemplate -n MyProject --namespace MyCompany.MyProject

# 卸载模板
dotnet new uninstall ./my-custom-template
```

### 8.2 分享自定义模板

#### 方式一：NuGet 包发布

```bash
# 打包为 .nupkg
dotnet pack ./src/Templates/Templates.csproj -c Release -o ./output

# 发布到 NuGet
dotnet nuget push output/MyCompany.Templates.1.0.0.nupkg \
  --source https://api.nuget.org/v3/index.json \
  --api-key YOUR_API_KEY
```

#### 方式二：GitHub 分享

```bash
# 用户可以通过以下命令从 GitHub 安装
dotnet new install https://github.com/username/templates.git
```

### 8.3 模板参数高级用法

#### 条件包含

```json
{
  "symbols": {
    "includeDocker": {
      "type": "parameter",
      "datatype": "bool",
      "defaultValue": "false"
    }
  },
  "sources": [
    {
      "modifiers": [
        {
          "condition": "(includeDocker)",
          "exclude": [
            "Dockerfile/**"
          ]
        }
      ]
    }
  ]
}
```

#### 多选参数

```json
{
  "symbols": {
    "framework": {
      "type": "parameter",
      "datatype": "choice",
      "choices": [
        {
          "choice": "net8.0",
          "description": ".NET 8.0"
        },
        {
          "choice": "net9.0",
          "description": ".NET 9.0 (Preview)"
        }
      ],
      "defaultValue": "net8.0"
    }
  }
}
```

---

## 九、推荐的项目起始模板组合

### 9.1 Web API 微服务

```bash
# 推荐的项目结构
mkdir MyMicroservice && cd MyMicroservice

# 1. 主 API 项目
dotnet new webapi -n MyMicroservice.Api --openapi true

# 2. 领域模型层
dotnet new classlib -n MyMicroservice.Domain

# 3. 应用服务层
dotnet new classlib -n MyMicroservice.Application

# 4. 基础设施层
dotnet new classlib -n MyMicroservice.Infrastructure

# 5. 单元测试
dotnet new xunit -n MyMicroservice.UnitTests

# 6. 集成测试
dotnet new xunit -n MyMicroservice.IntegrationTests

# 7. 创建解决方案并添加项目
dotnet new sln -n MyMicroservice
dotnet sln add **/*.csproj --in-folder

# 8. 添加项目引用
cd MyMicroservice.Application
dotnet add reference ../MyMicroservice.Domain/MyMicroservice.Domain.csproj

cd ../MyMicroservice.Infrastructure
dotnet add reference ../MyMicroservice.Application/MyMicroservice.Application.csproj

cd ../MyMicroservice.Api
dotnet add reference ../MyMicroservice.Application/MyMicroservice.Application.csproj
dotnet add reference ../MyMicroservice.Infrastructure/MyMicroservice.Infrastructure.csproj
```

### 9.2 Blazor 企业应用

```bash
mkdir EnterpriseApp && cd EnterpriseApp

# 1. Blazor Server 主项目
dotnet new blazorserver -n EnterpriseApp.Web --auth Individual

# 2. 组件库
dotnet new razorclasslib -n EnterpriseApp.Components

# 3. 业务逻辑层
dotnet new classlib -n EnterpriseApp.Core

# 4. 数据访问层
dotnet new classlib -n EnterpriseApp.Data

# 5. 测试
dotnet new xunit -n EnterpriseApp.Tests

# 组装解决方案
dotnet new sln -n EnterpriseApp
dotnet sln add **/*.csproj --in-folder
```

### 9.3 gRPC 服务

```bash
mkdir GrpcService && cd GrpcService

# 1. gRPC 服务
dotnet new grpc -n MyGrpcService

# 2. Protobuf 定义
dotnet new protobuf -n contracts -o ./Contracts

# 3. 客户端库
dotnew new classlib -n MyGrpc.Client

# 4. 测试
dotnet new xunit -n MyGrpc.Tests
```

---

## 十、常用场景速查表

### 10.1 快速启动各类项目

| 场景 | 命令 | 说明 |
|------|------|------|
| 最简单的 Web API | `dotnet new webapi -n Api` | 含 Swagger |
| 带 Auth 的 Web App | `dotnet new mvc -n Web --auth Individual` | Identity UI |
| Blazor Server | `dotnet new blazorserver -n Blazor` | 实时 UI |
| Blazor WASM 托管 | `dotnet new blazorwasm -n Spa --hosted` | 全栈方案 |
| 控制台工具 | `dotnew new console -n CliTool` | 工具开发 |
| 后台服务 | `dotnet new worker -n BackgroundSvc` | 定时任务 |
| Azure Function | `dotnet new azure-function -n Func` | 无服务器 |
| gRPC 服务 | `dotnet new grpc -n RpcSrv` | 高性能 RPC |
| 类库 | `dotnet new classlib -n Lib` | 代码复用 |
| 测试项目 | `dotnet new xunit -n Tests` | 单元测试 |

### 10.2 框架版本快速切换

```bash
# .NET 8.0 LTS
dotnet new webapi -n Api8 --framework net8.0

# .NET 9.0 STS
dotnet new webapi -n Api9 --framework net9.0

# .NET Standard 2.0（兼容性）
dotnet new classlib -n Lib --framework netstandard2.0

# 多目标框架
dotnet new classlib -n MultiLib
# 然后修改 csproj:
# <TargetFrameworks>net8.0;netstandard2.0</TargetFrameworks>
```

### 10.3 带完整配置的项目初始化

```bash
# 一键创建完整的 .NET 项目脚手架
create-dotnet-project() {
  local project_name=$1
  local type=${2:-"webapi"}

  # 创建主目录
  mkdir $project_name && cd $project_name

  # 创建项目
  dotnet new $type -n ${project_name}.Api --openapi true
  dotnet new classlib -n ${project_name}.Core
  dotnet new classlib -n ${project_name}.Infrastructure
  dotnet new xunit -n ${project_name}.Tests

  # 创建解决方案
  dotnet new sln -n $project_name
  dotnet sln add **/*.csproj --in-folder

  # 配置文件
  dotnet new gitignore
  dotnet new editorconfig
  dotnew new globaljson --sdk-version 8.0.400

  # 还原依赖
  dotnet restore

  echo "✓ 项目 $project_name 创建完成！"
}

# 使用示例
create-dotnet-project MyAwesomeApp webapi
```

---

## 十一、模板管理与维护

### 11.1 查看已安装模板

```bash
# 列出所有已安装模板
dotnet new --list

# 只显示自定义模板
dotnet new --list --columns "Short Name,Author,Language"

# 显示模板详细信息
dotnet new webapi --list
```

### 11.2 更新内置模板

```bash
# 更新 .NET SDK 会自动更新内置模板
dotnet sdk install 8.0.401

# 或更新到最新版本
dotnet sdk install latest
```

### 11.3 清理无用模板

```bash
# 查看所有已安装的自定义模板来源
dotnet new --show-all

# 卸载特定模板
dotnet new uninstall <path-or-nuget-id>

# 重置为仅内置模板
dotnet new --clear-alias
```

---

## 十二、最佳实践总结

### 12.1 模板选择原则

1. **简单优先**：能用简单模板就不用复杂模板
2. **按需添加**：先创建最小可行项目，再逐步添加功能
3. **保持一致**：团队统一使用相同的模板和参数
4. **版本锁定**：使用 `global.json` 锁定 SDK 版本
5. **模板定制**：将常用配置封装为团队模板

### 12.2 性能优化建议

- 使用 `--no-restore` 加速项目创建
- 对于大型解决方案，考虑使用 `.slnf` 过滤器文件
- 将常用模板缓存到本地以加速网络访问
- 使用 `--dry-run` 预览复杂模板的输出结果

### 12.3 团队协作建议

- 维护团队内部模板仓库
- 编写模板使用文档
- 建立 CI 流水线中的模板校验步骤
- 定期审查和更新模板以跟上 .NET 版本更新

---

## 附录：完整模板列表速查

| 分类 | 模板短名称 | 完整名称 |
|------|-----------|----------|
| **Web** | `webapp` | ASP.NET Core Web App |
| | `webapi` | ASP.NET Core Web API |
| | `mvc` | ASP.NET Core Web App (MVC) |
| | `blazorserver` | Blazor Server App |
| | `blazorwasm` | Blazor WebAssembly App |
| | `razorpages` | ASP.NET Core Web App (Razor Pages) |
| | `razorcomponent` | Razor Component |
| | `grpc` | gRPC Service |
| | `worker` | Worker Service |
| | `angular` | Angular + ASP.NET Core |
| | `react` | React + ASP.NET Core |
| | `vue` | Vue.js + ASP.NET Core |
| **类库** | `classlib` | Class Library |
| | `mstest` | MSTest Test Project |
| | `xunit` | xUnit Test Project |
| | `nunit` | NUnit Test Project |
| | `console` | Console Application |
| | `winforms` | Windows Forms App |
| | `wpf` | WPF Application |
| | `razorclasslib` | Razor Class Library |
| **配置** | `gitignore` | Git Ignore File |
| | `editorconfig` | EditorConfig File |
| | `sln` | Solution File |
| | `globaljson` | global.json File |
| | `nugetconfig` | NuGet Config |
| | `protocol` | Protocol Buffer Def |
| | `dockerfile` | Dockerfile |
| | `directory.build.props` | Directory.Build.Props |
| | `props` | MSBuild Props |
| | `targets` | MSBuild Targets |
| **Azure** | `azure-function` | Azure Functions |
| | `worker-grpc` | gRPC Worker Service |
| | `mvc-auth` | MVC with Authentication |
| | `webapp-auth` | Web App with Auth |
| | `razor-auth` | Razor Pages with Auth |
| | `blazorserver-auth` | Blazor Server with Auth |
| | `blazorwasm-auth` | Blazor WASM with Auth |
| | `webapi-auth` | Web API with Auth |
| **测试** | `xunit` | xUnit Test Project |
| | `nunit` | NUnit Test Project |
| | `mstest` | MSTest Test Project |
| | `razorclasslib` | Razor Class Library |
| | `console` | Console App (for testing) |
| | `web` | Web App (for testing) |
| **其他** | `console` | Console Application |
| | `solution` | Solution File |
| | `protobuf` | Protocol Buffer Def |
| | `grpc` | gRPC Service/Client |
| | `webconfig` | Web Config File |
| | `rulefile` | Analyzer Rule File |

掌握这些模板的使用方法，可以显著提升 .NET 项目的创建效率。根据具体需求选择合适的模板，并结合自定义模板能力，打造符合团队规范的开发流程。
