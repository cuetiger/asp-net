# VS Code 工作区配置完全指南

> 一个精心配置的 VS Code 工作区能显著提升开发效率，并确保团队成员之间的一致性。本文将深入讲解 VS Code 的配置体系，提供针对 ASP.NET 开发优化的完整配置方案。

---

## 目录

- [一、配置体系概述](#一配置体系概述)
- [二、推荐的 settings.json 完整配置](#二推荐的-settingsjson-完整配置)
- [三、tasks.json 任务配置详解](#三tasksjson-任务配置详解)
- [四、推荐的工作区结构](#四推荐的工作区结构)
- [五、多项目工作区配置](#五多项目工作区配置)
- [六、团队共享配置最佳实践](#六团队共享配置最佳实践)

---

## 一、配置体系概述

### 1.1 User Settings vs Workspace Settings

VS Code 采用分层配置系统，理解各层级的优先级是正确配置的基础。

```
┌─────────────────────────────────────────────┐
│           配置优先级（从高到低）              │
├─────────────────────────────────────────────┤
│                                             │
│  ① 工作区设置 (Workspace Settings)          │
│     位置: .vscode/settings.json             │
│     范围: 当前工作区                         │
│     特点: 可纳入 Git 版本控制                │
│                                             │
│  ② 用户设置 (User Settings)                 │
│     位置: %APPDATA%/Code/User/settings.json │
│     范围: 所有工作区（全局）                  │
│     特点: 个人偏好，不共享                   │
│                                             │
│  ③ 默认设置 (Default Settings)              │
│     位置: VS Code 内置                      │
│     范围: 所有用户                           │
│     特点: 不可修改                          │
│                                             │
└─────────────────────────────────────────────┘
```

#### 各层级适用场景对比

| 配置类型 | 存储位置 | 适用场景 | 是否纳入版本控制 |
|----------|----------|----------|-----------------|
| **User Settings** | `AppData/Code/User/` | 个人偏好（主题、字体、快捷键） | 否 |
| **Workspace Settings** | `.vscode/settings.json` | 项目规范（格式化规则、Linter） | 是 |
| **Folder Settings** | `.vscode/settings.json`（多根目录时） | 特定文件夹的覆盖配置 | 是 |

#### 推荐的配置分配原则

```
User Settings（个人偏好，不共享）：
├── 外观：颜色主题、文件图标主题、字体大小
├── 快捷键绑定：个人习惯的键位映射
├── 终端 Shell：个人偏好的终端类型
├── 账户同步：Settings Sync 配置
└── 编辑器行为：自动保存间隔等

Workspace Settings（团队规范，需共享）：
├── C#/代码风格：缩进、命名约定、格式化规则
├── 文件关联：.csproj/.razor 等文件类型
├── 搜索排除：bin/obj/node_modules 等
├── 任务配置：build/test/run 命令
├── 调试配置：launch.json
└── 扩展推荐：extensions.json
```

### 1.2 打开和编辑配置的方式

**方式1：GUI 设置界面**

```
操作路径：
- Windows/Linux: File > Preferences > Settings (或 Ctrl+,)
- Mac: Code > Preferences > Settings (或 Cmd+,)

在 GUI 中可以：
- 搜索设置项名称
- 查看 JSON 格式预览
- 区分 User 和 Workspace 设置
- 重置为默认值
```

**方式2：直接编辑 JSON**

```
快捷打开:
- Ctrl+Shift+P -> "Preferences: Open User Settings (JSON)"
- Ctrl+Shift+P -> "Preferences: Open Workspace Settings (JSON)"
```

**方式3：命令面板快速修改**

```bash
# 示例：切换某个设置
Ctrl+Shift+P -> "Preferences: Toggle Tab Visibility"

# 示例：按语言配置格式化器
Ctrl+Shift+P -> "Documents: Configure Default Formatter"
```

---

## 二、推荐的 settings.json 完整配置

以下是一个经过实战验证的 ASP.NET 开发工作区配置，涵盖约 150 行设置项。

### 2.1 编辑器基础配置

```json
{
  // ========================================
  // 第一部分：编辑器基础配置
  // ========================================

  // ---- 字体与外观 ----
  
  // 字体大小（建议 14px，兼顾清晰度和空间利用）
  "editor.fontSize": 14,
  
  // 字体家族（推荐支持连字的编程字体）
  "editor.fontFamily": "'Cascadia Code', 'Fira Code', 'JetBrains Mono', Consolas, monospace",
  
  // 启用字体连字（如 => 变成 ⇒，!= 变成 ≠）
  "editor.fontLigatures": true,
  
  // 行高（1.6 提高可读性）
  "editor.lineHeight": 1.6,
  
  // 字符间距（微调）
  "editor.letterSpacing": 0,

  // ---- 缩进与制表符 ----
  
  // Tab 大小（C# 标准使用 4 空格）
  "editor.tabSize": 4,
  
  // 使用空格替代 Tab（C# 编码规范要求）
  "editor.insertSpaces": true,
  
  // 关闭自动检测缩进（避免不一致）
  "editor.detectIndentation": false,
  
  // 自动缩进策略（仅对空行或特定模式生效）
  "editor.autoIndent": "full",

  // ---- 保存与格式化 ----
  
  // 自动保存策略：延迟 1 秒后自动保存
  "files.autoSave": "afterDelay",
  "files.autoSaveDelay": 1000,
  
  // 保存时自动格式化（核心功能！）
  "editor.formatOnSave": true,
  
  // 保存时自动移除尾随空格
  "files.trimTrailingWhitespace": true,
  
  // 保存时确保文件末尾有换行符
  "files.insertFinalNewline": true,
  
  // 默认格式化器（C# 项目使用内置格式化器）
  "editor.defaultFormatter": "ms-dotnettools.csharp",

  // ---- 行号与导航辅助 ----
  
  // 行号显示模式（relative 模式方便 Vim 用户跳转）
  "editor.lineNumbers": "on",
  
  // 启用 Minimap（代码缩略图）
  "editor.minimap.enabled": true,
  "editor.minimap.maxColumn": 120,
  "editor.minimap.renderCharacters": false,
  "editor.minimap.showSlider": "mouseover",

  // ---- 文本编辑行为 ----
  
  // 自动换行（on 表示视窗边界换行）
  "editor.wordWrap": "on",
  
  // 标尺线（提示代码宽度限制）
  "editor.rulers": [120],
  
  // 光标样式（line 为竖线，block 为方块）
  "editor.cursorStyle": "line",
  
  // 光标闪烁动画
  "editor.cursorBlinking": "smooth",
  
  // 选区圆角边框
  "editor.roundedSelection": true,
  
  // 平滑滚动
  "editor.smoothScrolling": true,
  
  // 链接悬停检测
  "editor.links": true,
  
  // 彩色括号配对
  "editor.bracketPairColorization.enabled": true,
  
  // 引导线（缩进参考线）
  "editor.guides.indentation": true,
  "editor.guides.bracketPairs": true,

  // ---- 智能感知增强 ----
  
  // 建议（补全）延迟时间（毫秒）
  "editor.quickSuggestionsDelay": 50,
  
  // 在注释中也触发建议
  "editor.quickComments": {
    "other": true,
    "comments": false,
    "strings": false
  },
  
  // Snippet 模式下 Tab 键优先插入下一个占位符
  "editor.tabCompletion": "on",
  
  // 建议栏位置（底部不遮挡代码）
  "editor.suggest.selectionMode": "first",
  "editor.suggest.showStatusBar": true,
  
  // 参数提示触发
  "editor.parameterHints.enabled": true,
}
```

### 2.2 C# 语言特定配置

```json
{
  // ========================================
  // 第二部分：C# / .NET 特定配置
  // ========================================

  // ---- OmniSharp 引擎配置 ----
  
  // 启用导入完成（输入类型名自动添加 using）
  "omnisharp.enableImportCompletion": true,
  
  // 启用 Roslyn 分析器（更强大的静态分析）
  "omnisharp.enableRoslynAnalyzers": true,
  
  // 只分析当前打开的文档（提升性能）
  "omnisharp.analyzeOpenDocumentsOnly": true,
  
  // 使用 .NET SDK 自带的 MSBuild（而非全局 Mono）
  "omnisharp.useGlobalMono": "never",
  
  // 启用 MSBuild 的增量构建
  "dotnet.server.useOmnisharp": false,

  // ---- C# 代码风格 ----
  
  // 启用 C# 格式化
  "csharp.format.enable": true,
  
  // C# 文件默认格式化器
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp",
    "editor.suggest.selectionMode": "first",
    "editor.inlayHints.enabled": "offUnlessPressed"
  },

  // ---- 分析器严重级别配置 ----
  
  // 自定义诊断规则的严重程度
  "dotnet.diagnosticSeverityRules": {
    // 未使用的成员警告
    "CS0067": "warning",      // 未使用的事件
    "CS0168": "warning",      // 未使用的局部变量
    "CS0219": "warning",      // 未使用的变量赋值
    "CS0414": "warning",      // 未使用的私有字段
    "CS0649": "suggestion",   // 从未赋值的字段
    
    // 可空引用类型相关
    "CS8618": "warning",      // 非空属性未初始化
    "CS8600": "warning",      // 可空到非空的转换
    "CS8602": "warning",      // 可能解引用空引用
    "CS8603": "warning",      // 可能返回 null
    "CS8604": "warning",      // 可能传入 null 参数
    "CS8619": "warning",      // 可空值类型的转换
    "CS8625": "warning",      // 不能将 null 字面量转换为非空引用
    "CS8714": "warning"       // 类型可能为 null
  },
  
  // EditorConfig 规则覆盖
  "csharp.inlayHints.parameters.enabled": true,
  "csharp.inlayhints.parameters.forLiteralParameters": true,
  "csharp.inlayhints.parameters.forIndexerParameters": true,
  "csharp.inlayhints.parameters.forObjectCreationParameters": true,
  "csharp.inlayhints.parameters.forOtherParameters": true,
  "csharp.inlayhints.types.enabled": true,

  // ---- IntelliCode 配置 ----
  
  // 启用所有模型的 AI 补全
  "vsintellicode.apiSurvey.models.csharp": ["*"],
  
  // 整行补全
  "vsintellicode.completion.enabled": true,
}
```

### 2.3 文件与搜索配置

```json
{
  // ========================================
  // 第三部分：文件与搜索配置
  // ========================================

  // ---- 文件排除规则 ----
  
  // 从资源管理器和搜索中排除这些文件/文件夹
  "files.exclude": {
    "**/bin": true,            // 编译输出
    "**/obj": true,            // 中间编译产物
    "**/node_modules": true,   // Node.js 依赖
    "**/.git": true,           // Git 目录
    "**/.vs": true,            // VS 用户文件
    "**/*.dll": true,          // 编译后的程序集
    "**/*.pdb": true,          // 调试符号
    "**/*.nupkg": true,        // NuGet 包
    "**/TestResults": true,    // 测试结果
    "**/PublishProfiles": true // 发布配置
  },
  
  // 仅从全局搜索中排除
  "search.exclude": {
    "**/bin": true,
    "**/obj": true,
    "**/node_modules": true,
    "**/.git": true,
    "**/*.min.js": true,
    "**/*.min.css": true,
    "**/Migrations/*.Designer.cs": true  // EF 迁移设计文件
  },

  // ---- 文件类型关联 ----
  
  // 将扩展名映射到语言模式
  "files.associations": {
    "*.cshtml": "html",         // Razor 视图
    "*.razor": "html",          // Razor 组件
    "*.cshtml.cs": "csharp",    // Razor 代码隐藏文件
    "*.csproj": "xml",          // 项目文件
    "*.config": "xml",          // 配置文件
    "*.props": "xml",           # MSBuild 属性
    "*.targets": "xml",         // MSBuild 目标
    "*.nuspec": "xml",          // NuGet 规范
    "*.globaljson": "json",     # .NET SDK 版本文件
    "*.ruleset": "xml",         // 代码分析规则集
    "*.pubxml": "xml"           # 发布配置
  },
  
  // 默认换行符（LF 兼容 Linux/Docker）
  "files.eol": "\n",
  
  // 文件编码（UTF-8）
  "files.encoding": "utf8",
  
  // 大文件阈值（超过此大小的文件不启用高级功能）
  "files.largeFileThreshold": 10 * 1024 * 1024, // 10MB
  
  // 监听文件变化（热重载支持）
  "files.watcherExclude": {
    "**/.git/objects/**": true,
    "**/.git/subtree-cache/**": true,
    "**/node_modules/**": true,
    "**/bin/**": true,
    "**/obj/**": true
  }
}
```

### 2.4 终端配置

```json
{
  // ========================================
  // 第四部分：集成终端配置
  // ========================================

  // ---- 终端基本设置 ----
  
  // 默认终端配置文件
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.profiles.windows": {
    "PowerShell": {
      "source": "PowerShell",
      "icon": "terminal-powershell",
      "args": ["-NoLogo"]
    },
    "Command Prompt": {
      "path": ["cmd.exe"],
      "icon": "terminal-cmd",
      "args": []
    },
    "Git Bash": {
      "source": "Git Bash"
    },
    "Developer Command Prompt for VS 2022": {
      "path": [
        "C:\\Program Files\\Microsoft Visual Studio\\2022\\Community\\Common7\\Tools\\VsDevCmd.bat"
      ],
      "args": [],
      "icon": "terminal-cmd"
    }
  },
  
  // 终端字体
  "terminal.integrated.fontSize": 13,
  "terminal.integrated.fontFamily": "'Cascadia Code PL', 'Cascadia Code', Consolas",
  "terminal.integrated.fontWeight": "normal",
  "terminal.integrated.fontWeightBold": "bold",
  
  // 光标样式
  "terminal.integrated.cursorStyle": "line",
  "terminal.integrated.cursorBlinking": true,
  
  // 滚动缓冲区（保留更多历史输出）
  "terminal.integrated.scrollback": 10000,
  
  // ---- 终端标签页 ----
  
  // 启用终端标签页
  "terminal.integrated.tabs.enabled": true,
  
  // 分离终端标题
  "terminal.integrated.tabs.separator": " \u2502 ",  // │ 分隔符
  "terminal.integrated.tabs.title": "${process}",
  
  // 新终端的工作目录
  "terminal.integrated.splitCwd": "initial",
  
  // ---- 终端交互行为 ----
  
  // 复制选中文本时自动复制到剪贴板
  "terminal.integrated.copyOnSelection": true,
  
  // 右键粘贴
  "terminal.integrated.rightClickBehavior": "paste",
  
  // 环境变量（可选）
  "terminal.integrated.env.windows": {
    "DOTNET_CLI_TELEMETRY_OPTOUT": "1",
    "ASPNETCORE_ENVIRONMENT": "Development"
  }
}
```

### 2.5 Git 与版本控制配置

```json
{
  // ========================================
  // 第五部分：Git 与版本控制配置
  // ========================================

  // ---- Git 基本行为 ----
  
  // 自动获取远程更新
  "git.autofetch": true,
  
  // 同步前不需要确认
  "git.confirmSync": false,
  
  // 允许没有暂存的提交
  "git.enableSmartCommit": true,
  
  // 输入框最大长度
  "git.inputValidationLength": 120,
  "git.inputValidationSubjectLength": 72,
  
  // 子模块检测限制
  "git.detectSubmodulesLimit": 10,
  
  // 忽略特定警告
  "git.ignoreMissingGitWarning": true,
  "git.ignoreLegacyWarning": true,
  
  // ---- GitLens 配置 ----
  
  // 当前行 Git 信息
  "gitlens.currentLine.enabled": true,
  
  // Code Lens 显示选项
  "gitlens.codeLens.authors.enabled": true,
  "gitlens.codeLens.recentChange.enabled": true,
  "gitlens.codeLens.recentChange.enabled": true,
  
  // 信息面板
  "gitlens.views.repositories.location": "sidebar",
  "gitlens.views.fileHistory.enabled": true,
  "gitlens.views.lineHistory.enabled": true,
  
  // ---- Git 图形视图 ----
  
  // Git Graph 设置
  "git-graph.repository.showRemoteNameToDifferentiateRemotesWithSameUrl": true,
  "git-graph.graph.showCommitsOnlyReferencedByAnotherBranch": false,
  "git-graph.timeline.showDate": true,
  "git-graph.timeline.dateFormat": "yyyy-MM-dd HH:mm:ss"
}
```

### 2.6 扩展与外观配置

```json
{
  // ========================================
  // 第六部分：扩展与外观配置
  // ========================================

  // ---- 外观主题 ----
  
  // 文件图标主题
  "workbench.iconTheme": "material-icon-theme",
  
  // 颜色主题
  "workbench.colorTheme": "One Dark Pro",
  
  // 标题栏样式（native 使用系统原生标题栏）
  "window.titleBarStyle": "native",
  
  // 窗口标题格式
  "window.title": "${dirty}${activeEditorMedium}${separator}${rootName}${separator}${profileName}",
  
  // ---- 功能开关 ----
  
  // 启用面包屑导航
  "breadcrumbs.enabled": true,
  
  // 欢迎页面（关闭以加快启动）
  "workbench.startupEditor": "none",
  
  // 自动更新（由组织统一管理时关闭）
  "update.mode": "none",
  
  // 编辑器布局
  "workbench.editor.openPositioning": "right",
  "workbench.editor.highlightModifiedTabs": true,
  "workbench.editor.wrapTabs": true,
  
  // ---- Error Lens 配置 ----
  
  "errorLens.delay": 0,
  "errorLens.errorMessageEnabled": true,
  "errorLens.warningMessageEnabled": true,
  "errorLens.infoMessageEnabled": true,
  "errorLens.hintMessageEnabled": true,
  "errorLens.errorGutterIconEnabled": true,
  "errorLens.warningGutterIconEnabled": true,
  "errorLens.infoGutterIconEnabled": true,
  "errorLens.hintGutterIconEnabled": true,
  "errorLens.fontStyleItalic": true,
  "errorLens.fontSizeRatio": 0.85,
  "errorLens.messageMaxChars": 150,
  
  // ---- TODO Tree 配置 ----
  
  "todo-tree.general.tags": [
    "TODO", "FIXME", "BUG", "HACK", 
    "NOTE", "REVIEW", "XXX", "OPTIMIZE"
  ],
  "todo-tree.highlights.defaultHighlight": {
    "gutterIcon": true
  },
  "todo-tree.highlights.customHighlight": {
    "TODO": {
      "icon": "checklist",
      "foreground": "#fff",
      "background": "#ffbd2a",
      "iconColour": "#ffbd2a"
    },
    "FIXME": {
      "icon": "alert",
      "foreground": "#fff",
      "background": "#f06292",
      "iconColour": "#f06292"
    },
    "BUG": {
      "icon": "bug",
      "foreground": "#fff",
      "background": "#ef5350",
      "iconColour": "#ef5350"
    },
    "NOTE": {
      "icon": "note",
      "foreground": "#fff",
      "background": "#42a5f5",
      "iconColour": "#42a5f5"
    }
  }
}
```

---

## 三、tasks.json 任务配置详解

`tasks.json` 定义了可以在 VS Code 中执行的自定义任务，通常用于构建、测试和运行 .NET 应用程序。

### 3.1 基础 tasks.json 结构

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      // 任务定义...
    }
  ]
}
```

### 3.2 ASP.NET Core 常用任务配置

以下是完整的 `tasks.json` 配置示例：

```json
{
  "$schema": "https://raw.githubusercontent.com/microsoft/vscode/main/src/vs/workbench/contrib/tasks/common/taskSchema.json",
  "version": "2.0.0",
  "tasks": [
    // ==================== 构建任务 ====================
    
    {
      "label": "build",
      "command": "dotnet",
      "type": "process",
      "args": ["build", "${workspaceFolder}/MyApp.sln"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": true,
        "clear": false
      },
      "problemMatcher": "$msCompile"
    },
    
    // Debug 构建
    {
      "label": "build-debug",
      "command": "dotnet",
      "type": "process",
      "args": [
        "build",
        "${workspaceFolder}/MyApp.sln",
        "-c", "Debug",
        "--no-restore"
      ],
      "group": "build",
      "problemMatcher": "$msCompile",
      "dependsOn": "restore"
    },
    
    // Release 构建
    {
      "label": "build-release",
      "command": "dotnet",
      "type": "process",
      "args": [
        "build",
        "${workspaceFolder}/MyApp.sln",
        "-c", "Release",
        "--no-restore"
      ],
      "group": "build",
      "problemMatcher": "$msCompile",
      "dependsOn": "restore"
    },

    // ==================== 还原依赖任务 ====================
    
    {
      "label": "restore",
      "command": "dotnet",
      "type": "process",
      "args": ["restore", "${workspaceFolder}/MyApp.sln"],
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": true
      },
      "problemMatcher": []
    },

    // ==================== 测试任务 ====================
    
    {
      "label": "test",
      "command": "dotnet",
      "type": "process",
      "args": [
        "test",
        "${workspaceFolder}/MyApp.sln",
        "--no-build",
        "--verbosity", "normal"
      ],
      "group": {
        "kind": "test",
        "isDefault": true
      },
      "problemMatcher": "$msCompile",
      "dependsOn": ["build-debug"]
    },
    
    // 带覆盖率的测试
    {
      "label": "test-with-coverage",
      "command": "dotnet",
      "type": "process",
      "args": [
        "test",
        "${workspaceFolder}/MyApp.sln",
        "/p:CollectCoverage=true",
        "/p:CoverletOutputFormat=opencover",
        "/p:CoverletOutput=./coverage/",
        "--no-build",
        "--verbosity", "normal"
      ],
      "group": "test",
      "problemMatcher": "$msCompile",
      "dependsOn": ["build-debug"]
    },
    
    // 只运行特定的测试类
    {
      "label": "test-unit",
      "command": "dotnet",
      "type": "process",
      "args": [
        "test",
        "${workspaceFolder}/tests/UnitTests/",
        "--filter", "FullyQualifiedName~UnitTests",
        "--no-build",
        "--verbosity", "minimal"
      ],
      "group": "test",
      "problemMatcher": "$msCompile"
    },
    
    // 只运行集成测试
    {
      "label": "test-integration",
      "command": "dotnet",
      "type": "process",
      "args": [
        "test",
        "${workspaceFolder}/tests/IntegrationTests/",
        "--filter", "FullyQualifiedName~IntegrationTests",
        "--no-build",
        "--verbosity", "minimal"
      ],
      "group": "test",
      "problemMatcher": "$msCompile"
    },

    // ==================== 运行任务 ====================
    
    {
      "label": "run",
      "command": "dotnet",
      "type": "process",
      "args": [
        "run",
        "--project", "${workspaceFolder}/src/MyApp.Api/MyApp.Api.csproj",
        "--no-build",
        "--launch-profile", "Development"
      ],
      "group": {
        "kind": "test",
        "isDefault": false
      },
      "problemMatcher": "$msCompile",
      "dependsOn": ["build-debug"],
      "isBackground": true,
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "dedicated",
        "showReuseMessage": false,
        "clear": true
      }
    },

    // ==================== 发布任务 ====================
    
    {
      "label": "publish",
      "command": "dotnet",
      "type": "process",
      "args": [
        "publish",
        "${workspaceFolder}/src/MyApp.Api/MyApp.Api.csproj",
        "-c", "Release",
        "-o", "${workspaceFolder}/publish",
        "--self-contained",
        "-r", "win-x64",
        "/p:PublishSingleFile=true"
      ],
      "problemMatcher": "$msCompile",
      "dependsOn": ["build-release"]
    },
    
    // Docker 发布
    {
      "label": "publish-docker",
      "command": "docker",
      "type": "process",
      "args": [
        "build",
        "-t", "myapp-api:${input:imageTag}",
        "-f", "${workspaceFolder}/Dockerfile",
        "."
      ],
      "options": {
        "cwd": "${workspaceFolder}"
      }
    },

    // ==================== EF Core 任务 ====================
    
    {
      "label": "ef-migrations-add",
      "command": "dotnet",
      "type": "process",
      "args": [
        "ef", "migrations", "add", "${input:migrationName}",
        "--project", "${workspaceFolder}/src/Infrastructure/Data/",
        "--startup-project", "${workspaceFolder}/src/MyApp.Api/"
      ],
      "problemMatcher": []
    },
    
    {
      "label": "ef-database-update",
      "command": "dotnet",
      "type": "process",
      "args": [
        "ef", "database", "update",
        "--project", "${workspaceFolder}/src/Infrastructure/Data/",
        "--startup-project", "${workspaceFolder}/src/MyApp.Api/"
      ],
      "problemMatcher": []
    },
    
    {
      "label": "ef-database-drop",
      "command": "dotnet",
      "type": "shell",
      "args": [
        "ef", "database", "drop", "--force",
        "--project", "${workspaceFolder}/src/Infrastructure/Data/",
        "--startup-project", "${workspaceFolder}/src/MyApp.Api/"
      ],
      "problemMatcher": [],
      "promptOnClose": true
    },

    // ==================== 代码质量任务 ====================
    
    {
      "label": "format-code",
      "command": "dotnet",
      "type": "process",
      "args": [
        "format", "style",
        "${workspaceFolder}/MyApp.sln",
        "--severity", "warn",
        "--verify-no-changes"
      ],
      "problemMatcher": []
    },
    
    {
      "label": "analyze-code",
      "command": "dotnet",
      "type": "process",
      "args": [
        "format", "analyzers",
        "${workspaceFolder}/MyApp.sln"
      ],
      "problemMatcher": []
    }
  ],
  
  // ==================== 输入变量定义 ====================
  
  "inputs": [
    {
      "id": "migrationName",
      "type": "promptString",
      "description": "请输入迁移名称（如 AddUserTable）",
      "default": "InitialCreate"
    },
    {
      "id": "imageTag",
      "type": "promptString",
      "description": "请输入 Docker 镜像标签（如 latest 或 v1.0.0）",
      "default": "latest"
    }
  ]
}
```

### 3.3 任务运行方式

```bash
# 方式1：通过终端菜单
Terminal -> Run Task -> 选择任务名

# 方式2：通过命令面板
Ctrl+Shift+P -> Tasks: Run Task -> 选择任务

# 方式3：通过快捷键绑定（自定义）
# Ctrl+Shift+B 执行默认 build 任务

# 方式4：在 launch.json 的 preLaunchTask 中引用
{
  "preLaunchTask": "build-debug"
}
```

### 3.4 Problem Matcher 说明

Problem Matcher 用于解析任务的输出并将其转换为 VS Code 可以识别的问题列表：

| Matcher 名称 | 适用场景 |
|-------------|---------|
| `$msCompile` | dotnet build/test 输出（CSC 错误格式） |
| `$msCompile-fullframe` | 更详细的编译错误信息 |
| `$eslint-stylish` | ESLint 输出 |
| `$go` | Go 语言编译输出 |
| `$tsc` | TypeScript 编译输出 |

---

## 四、推荐的工作区结构

一个良好的工作区结构能让项目管理和团队协作更加高效。

### 4.1 标准 ASP.NET 解决方案结构

```
MySolution/
├── .vscode/                        # VS Code 工作区配置（纳入版本控制）
│   ├── settings.json               # 工作区设置
│   ├── tasks.json                  # 构建任务定义
│   ├── launch.json                 # 调试启动配置
│   └── extensions.json             # 推荐扩展列表
│
├── src/                            # 源代码目录
│   ├── MyApp.Api/                  # API 主项目（ASP.NET Core Web API）
│   │   ├── Controllers/
│   │   ├── Services/
│   │   ├── Models/
│   │   ├── DTOs/
│   │   ├── Middleware/
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   └── MyApp.Api.csproj
│   │
│   ├── MyApp.Domain/               # 领域模型（领域驱动设计）
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Enums/
│   │   ├── Interfaces/
│   │   └── MyApp.Domain.csproj
│   │
│   ├── MyApp.Application/          # 应用服务层
│   │   ├── Interfaces/
│   │   ├── Services/
│   │   ├── DTOs/
│   │   ├── Mappings/
│   │   └── MyApp.Application.csproj
│   │
│   ├── MyApp.Infrastructure/       # 基础设施层
│   │   ├── Data/
│   │   │   ├── DbContext/
│   │   │   ├── Migrations/
│   │   │   ├── Repositories/
│   │   │   └── Configurations/
│   │   ├── ExternalServices/
│   │   ├── Identity/
│   │   └── MyApp.Infrastructure.csproj
│   │
│   └── MyApp.Web/                  # Blazor/Razor Pages 前端（可选）
│       ├── Pages/
│       ├── Shared/
│       ├── wwwroot/
│       └── MyApp.Web.csproj
│
├── tests/                          # 测试目录
│   ├── UnitTests/                  # 单元测试
│   │   ├── Services/
│   │   ├── Helpers/
│   │   └── UnitTests.csproj
│   │
│   ├── IntegrationTests/           # 集成测试
│   │   ├── Fixtures/
│   │   ├── Helpers/
│   │   └── IntegrationTests.csproj
│   │
│   └── PerformanceTests/           # 性能测试（可选）
│       └── PerformanceTests.csproj
│
├── docs/                           # 项目文档（可选）
│   ├── api/                        # API 文档
│   └── architecture/               # 架构文档
│
├── scripts/                        # 构建和部署脚本
│   ├── build.ps1                   # PowerShell 构建脚本
│   ├── deploy.ps1                  # 部署脚本
│   └── docker-compose.yml          # 本地开发容器编排
│
├── .github/                        # GitHub 配置（如使用 GitHub Actions）
│   └── workflows/
│       ├── ci.yml                  # CI 流水线
│       └── cd.yml                  # CD 流水线
│
├── .dockerignore                   # Docker 忽略文件
├── .gitignore                      # Git 忽略文件
├── .editorconfig                   # 编辑器配置
├── global.json                     # .NET SDK 版本锁定
├── Directory.Build.props           # 共享 MSBuild 属性
├── Directory.Build.targets         # 共享 MSBuild 目标
├── nuget.config                    # NuGet 源配置
├── MySolution.sln                  # 解决方案文件
├── README.md                       # 项目说明
└── LICENSE                         # 许可证
```

### 4.2 .vscode/extensions.json 团队推荐扩展

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=827846
  // for the documentation about the extensions.json format
  "recommendations": [
    // C# 开发核心
    "ms-dotnettools.csdevkit",
    "ms-dotnettools.vscodeintellicode",
    "jchannon.csharpextensions",
    
    // Web 开发
    "ecmel.vscode-html-css",
    "ritwickdey.liveserver",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "rangav.thunder-client",
    
    // Git 集成
    "eamodio.gitlens",
    "mhutchie.git-graph",
    "donjayamanne.githistory",
    
    // 代码质量
    "usernamehw.errorlens",
    "streetsidesoftware.code-spell-checker",
    "shardulm94.trailing-spaces",
    
    // 效率工具
    "PKief.material-icon-theme",
    "zhuangtongfa.material-theme",
    "wayou.todo-tree",
    
    // Docker
    "ms-azuretools.vscode-docker",
    
    // 数据库
    "ms-mssql.mssql",
    "cweijan.vscode-redis"
  ]
}
```

当团队成员首次打开项目时，VS Code 会弹出提示安装这些推荐扩展。

---

## 五、多项目工作区配置

对于大型解决方案或微服务架构，可能需要管理多个相关项目。

### 5.1 创建多根目录工作区

**方法1：通过 UI 操作**

```
File -> Add Folder to Workspace... -> 选择要添加的文件夹
File -> Save Workspace As... -> 保存为 .code-workspace 文件
```

**方法2：手动创建 workspace 文件**

```json
{
  "folders": [
    {
      "name": "API Backend",
      "path": "../my-solution/src/MyApp.Api"
    },
    {
      "name": "Frontend App",
      "path": "../my-solution/src/MyApp.Web"
    },
    {
      "name": "Shared Library",
      "path": "../my-solution/src/MyApp.Infrastructure"
    },
    {
      "name": "Tests",
      "path": "../my-solution/tests"
    }
  ],
  "settings": {
    // 工作区级别的设置会覆盖各个文件夹中的设置
    "files.exclude": {
      "**/bin": true,
      "**/obj": true
    }
  },
  "launch": {
    // 多目标调试配置
    "configurations": [...],
    "compounds": [...]
  },
  "tasks": {
    "version": "2.0.0",
    "tasks": [...]
  }
}
```

### 5.2 微服务工作区示例

```json
{
  "folders": [
    { "path": "./services/api-gateway", "name": "API Gateway" },
    { "path": "./services/user-service", "name": "User Service" },
    { "path": "./services/order-service", "name": "Order Service" },
    { "path": "./services/product-service", "name": "Product Service" },
    { "path": "./shared/kernel", "name": "Shared Kernel" },
    { "path": "./infrastructure/docker", "name": "Docker Config" }
  ],
  "settings": {
    "files.exclude": {
      "**/bin": true,
      "**/obj": true,
      "**/node_modules": true
    },
    "search.include": {
      "**/*.cs": true,
      "**/*.ts": true,
      "**/*.json": true
    },
    "dotnet.diagnosticSeverityRules": {
      "CS8618": "warning",
      "CS8600": "warning"
    }
  },
  "launch": {
    "configurations": [
      {
        "name": "Launch API Gateway",
        "type": "coreclr",
        "request": "launch",
        "program": "${workspaceFolder:api-gateway}/bin/Debug/net8.0/Gateway.dll",
        "cwd": "${workspaceFolder:api-gateway}",
        "env": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      },
      {
        "name": "Launch User Service",
        "type": "coreclr",
        "request": "launch",
        "program": "${workspaceFolder:user-service}/bin/Debug/net8.0/UserService.dll",
        "cwd": "${workspaceFolder:user-service}",
        "env": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      },
      {
        "name": "Launch Order Service",
        "type": "coreclr",
        "request": "launch",
        "program": "${workspaceFolder:order-service}/bin/Debug/net8.0/OrderService.dll",
        "cwd": "${workspaceFolder:order-service}",
        "env": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      }
    ],
    "compounds": [
      {
        "name": "Launch All Services",
        "configurations": [
          "Launch API Gateway",
          "Launch User Service",
          "Launch Order Service"
        ],
        "stopAll": true,
        "preLaunchTask": "build-all-services"
      }
    ]
  }
}
```

---

## 六、团队共享配置最佳实践

### 6.1 应该纳入版本控制的文件

```
.vscode/
├── settings.json          ✅ 应纳入版本控制（团队编码规范）
├── tasks.json             ✅ 应纳入版本控制（统一的构建流程）
├── launch.json            ✅ 应纳入版本控制（调试配置）
├── extensions.json        ✅ 应纳入版本控制（推荐扩展列表）
└── code-workspace.json    ⚠️ 可选（多人共用同一工作区时）

其他应纳入版本控制的配置文件：
├── .editorconfig          ✅ 编辑器通用配置（跨 IDE 兼容）
├── global.json            ✅ .NET SDK 版本约束
├── Directory.Build.props  ✅ 共享构建属性
└── .gitignore             ✅ Git 忽略规则
```

### 6.2 不应纳入版本控制的内容

```
# .gitignore 中应包含：
.vscode/
  !.vscode/settings.json
  !.vscode/tasks.json
  !.vscode/launch.json
  !.vscode/extensions.json
  !.vscode/code-workspace.json
  *.code-workspace         # 个人工作区布局
  launch.gsg               # 断点状态（个人）

# 用户个人配置不应共享：
- 个人主题/字体偏好
- 个人快捷键绑定
- 个人终端配置
- 个人扩展安装记录（除 recommendations 外）
```

### 6.3 团队配置管理流程

```
新成员加入时的配置同步流程：

1. 克隆仓库
   git clone <repository-url>

2. 用 VS Code 打开项目文件夹
   code .

3. VS Code 自动检测 .vscode/ 目录并应用配置

4. 弹出推荐扩展安装提示
   → 安装所有推荐扩展

5. 运行首次构建验证环境
   Terminal -> Run Task -> build

6. （可选）个人偏好设置
   Ctrl+, 打开 User Settings 进行个性化调整
```

### 6.4 配置冲突解决策略

```
场景1：团队成员对缩进有不同偏好
解决：在 settings.json 中强制统一
  "editor.tabSize": 4,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false

场景2：Windows/Mac/Linux 混合开发
解决：统一换行符和编码
  "files.eol": "\n",
  "files.encoding": "utf8"

场景3：不同项目的特殊需求
解决：使用 Folder Settings（多根目录工作区）
  每个 folder 可以有自己的 settings 覆盖

场景4：扩展版本兼容问题
解决：固定 extensions.json 并定期审查
  定期检查扩展是否有破坏性更新
```

### 6.5 配置维护清单

```
每季度检查项：

□ 审查 settings.json 是否有过时配置
□ 更新 extensions.json（新增/移除扩展）
□ 测试 tasks.json 中的所有任务是否正常工作
□ 验证 launch.json 的调试配置
□ 检查 .editorconfig 规则是否仍然适用
□ 更新 global.json 中的 SDK 版本
□ 清理不再需要的排除规则
□ 团队会议讨论是否需要新增配置项
```

---

## 总结

一个良好配置的 VS Code 工作区能够带来以下收益：

**效率提升**
- 一键构建、测试、发布
- 自动格式化和代码检查
- 智能补全和重构

**一致性保障**
- 团队统一的编码风格
- 相同的构建和调试流程
- 减少因环境差异导致的问题

**协作便利**
- 新成员快速搭建开发环境
- 配置即文档，减少沟通成本
- 版本控制保证配置的可追溯性

本文提供的完整配置方案已经过实际项目验证，你可以根据自己项目的具体需求进行调整和定制。
