# VSCode安装与配置

## 📚 学习目标

- 了解VSCode（Visual Studio Code）是什么以及为什么选择它
- 掌握VSCode的下载和安装流程
- 学会安装中文语言包，让界面更友好
- 掌握基本设置和常用快捷键，提高开发效率
- 配置适合.NET开发的编辑器环境

## 🔧 前置知识

在开始本教程前，请确保已完成：
- [x] [Windows环境准备](./01-Windows环境准备.md) - 确认电脑配置符合要求
- [x] [安装 .NET SDK](./02-安装.NET-SDK.md) - 已成功安装并验证.NET SDK

---

## 💡 核心内容

### 一、什么是VSCode？

**VSCode** = **V**isual **S**tudio **Code**（可视化工作室代码）

VSCode是微软推出的一款**免费、开源、轻量级**的代码编辑器。它虽然名字里有"Visual Studio"，但和我们后面要学的完整版Visual Studio IDE是完全不同的产品。

> **🎯 类比理解：**
> - **VSCode** 就像一把瑞士军刀——轻便、多功能、随身携带
> - **Visual Studio (完整版)** 就像整个工作台——功能全面但体积庞大
>
> 对于初学者来说，VSCode更容易上手，启动速度快，而且完全免费！

#### 为什么选择VSCode？

| 特点 | 说明 |
|------|------|
| ✅ **完全免费** | 不需要付费，无功能限制 |
| 🚀 **启动迅速** | 几秒钟就能打开 |
| 🎨 **界面美观** | 现代化UI设计 |
| 🔌 **扩展丰富** | 海量插件满足各种需求 |
| 💪 **功能强大** | 代码补全、调试、Git集成等 |
| 🌍 **跨平台** | Windows/Mac/Linux都能用 |
| 👥 **社区活跃** | 遇到问题容易找到解决方案 |

### 二、下载 VSCode

#### 步骤1：访问官网

打开浏览器，访问VSCode官方网站：
🔗 **https://code.visualstudio.com/**

> [!tip] 📸 操作提示
> 访问 https://code.visualstudio.com/ ，点击页面上的 **下载** 按钮

#### 步骤2：下载安装包

1. 在首页你会看到一个醒目的 **"Download for Windows"** 按钮（通常是绿色或蓝色）
2. 点击按钮开始下载
3. 文件会自动下载到你的 **"下载"** 文件夹

> [!tip] 📸 操作提示
> 点击 **Download for Windows** 按钮下载 System Installer 版本

**下载文件信息：**
- 文件名类似：`VSCodeSetup-x64-1.85.x.exe`
- 文件大小：约80-100MB
- 如果你是ARM架构的电脑（如Surface Pro X），选择"ARM64"版本

### 三、安装 VSCode

#### 安装步骤详解

**步骤1：运行安装程序**

1. 打开"下载"文件夹
2. 找到 `VSCodeSetup-x64-*.exe` 文件
3. **双击运行**
4. 如果弹出用户账户控制提示，点击 **"是"**

> [!tip] 📸 操作提示
> 运行下载的 `.exe` 安装包，点击 **"下一步"**

**步骤2：同意许可协议**

1. 勾选 **"我接受协议"**
2. 点击 **"下一步"**

> [!tip] 📸 操作提示
> 勾选 **"我同意协议"**，点击 **"下一步"**

**步骤3：选择安装位置**

- **默认路径：** `C:\Users\你的用户名\AppData\Local\Programs\Microsoft VS Code`
- **建议：** 保持默认即可（除非C盘空间紧张）
- 点击 **"下一步"**

> [!tip] 📸 操作提示
> 建议保持默认路径 `C:\Users\用户名\AppData\Local\Programs\Microsoft VS Code`

**步骤4：选择开始菜单文件夹**

- 默认名称：`Visual Studio Code`
- 直接点击 **"下一步"**

**步骤5：选择附加任务（重要！）⭐**

这一步建议勾选以下选项：

```
☑ 添加到PATH（重启后可用命令行启动）
☑ 添加到桌面快捷方式（方便快速打开）
☑ 添加到资源管理器上下文菜单（右键菜单可以"用VSCode打开"）
☑ 注册为受支持的文件类型的编辑器
```

> [!tip] 📸 操作提示
> 推荐全部勾选：**"添加到PATH"**、**"桌面快捷方式"**、**"资源管理器右键菜单"**

> **💡 小贴士：** 特别是 **"添加到PATH"** 和 **"资源管理器上下文菜单"** 这两个选项非常实用，强烈建议勾选！

**步骤6：准备安装**

点击 **"安装"** 按钮，开始安装过程。

**步骤7：等待安装完成**

安装通常只需要1-2分钟，你会看到进度条：

> [!tip] 📸 操作提示
> 等待进度条完成，通常需要1-2分钟

**步骤8：安装完成**

看到以下界面说明安装成功了！
- ☑️ 勾选 **"启动 Visual Studio Code"**
- 点击 **"完成"** 按钮

> [!tip] 📸 操作提示
> 点击 **"完成"** 按钮，可勾选 **"立即运行 VS Code"**

### 四、首次启动与初始设置

#### 第一次打开VSCode

安装完成后，VSCode会自动启动（如果你勾选了"启动"选项）。你将看到：

> [!tip] 📸 操作提示
> 首次启动会显示 Welcome 页面，可选择颜色主题

欢迎界面包含：
- **Start（开始）** - 最近打开的文件/文件夹
- **Recent（最近）** - 历史记录
- **Help（帮助）** - 学习资源和文档链接

### 五、安装中文语言包（强烈推荐）

虽然英文界面也能用，但中文界面会让初学者更舒适。让我们来安装官方中文语言包：

#### 方法1：通过扩展面板安装（推荐）

**步骤1：打开扩展面板**

点击左侧边栏的 **扩展图标** （看起来像四个方块，或者按快捷键 `Ctrl + Shift + X`）

> [!tip] 📸 操作提示
> 点击左侧边栏 **方块图标**（或按 `Ctrl+Shift+X`）打开扩展面板

**步骤2：搜索中文语言包**

1. 在搜索框输入：`Chinese`
2. 找到 **"Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code"**
3. 作者应该是 **Microsoft**（微软官方出品）
4. 点击 **"Install"** 安装按钮

> [!tip] 📸 操作提示
> 在搜索框输入 **"Chinese"**，找到 **"Chinese (Simplified)"** 包，点击 **Install**

**步骤3：切换语言**

安装完成后：
1. VSCode会弹出提示：**"Restart to switch language?"**（是否重启以切换语言？）
2. 点击 **"Restart"**（重启）
3. VSCode会自动重启，界面变成中文！

> [!tip] 📸 操作提示
> 安装完成后右下角弹出提示，点击 **"Restart in Chinese"** 重启

#### 方法2：通过命令面板安装

1. 按 `Ctrl + Shift + P` 打开命令面板
2. 输入：`Configure Display Language`
3. 选择 **"Install Additional Languages..."**
4. 搜索并安装中文包

### 六、基本设置优化

为了让开发体验更好，建议进行以下基础设置：

#### 打开设置界面的方法

- 方法1：点击左下角 ⚙️ 齿轮图标 → **设置**
- 方法2：使用快捷键 `Ctrl + ,`（逗号）
- 方法3：命令面板输入 `Preferences: Open Settings`

> [!tip] 📸 操作提示
> 按 `Ctrl+,` 打开设置，或通过 **文件 → 首选项 → 设置**

#### 推荐的基础设置

在设置界面中搜索并修改以下配置（或者直接编辑settings.json文件）：

##### 1. 字体大小调整

```json
// 设置编辑器字体大小
"editor.fontSize": 14
```

根据你的显示器大小和个人喜好调整，推荐 **14-16px**。

##### 2. 自动保存

```bash
# 设置方式：搜索 "auto save"
# 选择 "afterDelay"（延迟后自动保存）
"files.autoSave": "afterDelay"
```

这样就不怕忘记按 Ctrl+S 保存了！

##### 3. 格式化保存

```bash
# 搜索 "format on save"
# 勾选开启
"editor.formatOnSave": true
```

每次保存时自动格式化代码，保持代码整洁。

##### 4. Word Wrap（自动换行）

```bash
# 搜索 "word wrap"
# 选择 "on" 开启
"editor.wordWrap": "on"
```

长代码行自动换行，不用左右滚动查看。

##### 5. Minimap（代码缩略图）

```bash
# 搜索 "minimap"
# 可以关闭以节省空间（可选）
"editor.minimap.enabled": false
```

右侧的代码缩略图，觉得碍眼可以关掉。

#### 如何编辑 settings.json

1. 打开设置界面（`Ctrl + ,`）
2. 点击右上角的 **{} 图标**（打开JSON编辑器）
3. 将上面的配置复制进去
4. 按 `Ctrl + S` 保存

> [!tip] 📸 操作提示
> 在设置右上角点击 **"打开JSON"** 图标，编辑 `settings.json`

### 七、必备快捷键

掌握这些快捷键能大大提高你的效率！

| 快捷键 | 功能 | 使用频率 |
|--------|------|----------|
| `Ctrl + S` | 保存文件 | ★★★★★ |
| `Ctrl + Z` | 撤销 | ★★★★★ |
| `Ctrl + Y` / `Ctrl + Shift + Z` | 重做 | ★★★★☆ |
| `Ctrl + C` | 复制 | ★★★★★ |
| `Ctrl + X` | 剪切 | ★★★★★ |
| `Ctrl + V` | 粘贴 | ★★★★★ |
| `Ctrl + F` | 查找 | ★★★★★ |
| `Ctrl + H` | 替换 | ★★★★☆ |
| `Ctrl + G` | 跳转到指定行号 | ★★★★☆ |
| `Ctrl + P` | 快速打开文件 | ★★★★★ |
| `Ctrl + Shift + P` | 命令面板 | ★★★★★ |
| `Ctrl + B` | 显示/隐藏侧边栏 | ★★★★☆ |
| `Ctrl + ~` | 打开终端 | ★★★★★ |
| `Ctrl + W` | 关闭当前标签页 | ★★★★☆ |
| `Ctrl + Tab` | 切换标签页 | ★★★★★ |
| `Alt + ↑/↓` | 移动当前行 | ★★★☆☆ |
| `Shift + Alt + ↑/↓` | 复制当前行 | ★★★☆☆ |
| `Ctrl + /` | 注释/取消注释 | ★★★★★ |
| `Shift + Alt + A` | 块注释 | ★★★☆☆ |

> **💡 新手建议：** 不需要一次记住所有快捷键！先掌握最常用的几个（带五颗星的），其他的在使用过程中慢慢熟悉。

---

## 🛠️ 代码示例

### 示例1：创建并编辑第一个文件

让我们用VSCode创建一个简单的HTML文件来测试：

1. **创建新文件：** `Ctrl + N`
2. **保存文件：** `Ctrl + S`，命名为 `hello.html`
3. **输入以下代码：**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>我的第一个网页</title>
</head>
<body>
    <h1>Hello, VSCode!</h1>
    <p>这是我的第一个网页文件。</p>
</body>
</html>
```

4. 你会发现VSCode提供了：
   - **语法高亮** - 不同颜色的代码
   - **智能提示** - 输入时自动补全
   - **代码折叠** - 左侧可以收起代码块

> [!tip] 📸 操作提示
> 新建 `.html` 文件后自动触发语法高亮，标签会以不同颜色显示

### 示例2：使用内置终端

VSCode集成了终端，可以直接在里面执行命令：

1. 按 `Ctrl + ~` （反引号键，通常在Esc下方）打开终端
2. 或者点击菜单：**终端 → 新建终端**
3. 尝试输入之前学过的命令：

```powershell
# 检查.NET SDK版本
dotnet --version

# 查看当前目录
dir

# 列出文件
ls
```

> [!tip] 📸 操作提示
> 按 `Ctrl+~` 打开集成终端，可直接输入命令

---

## 📝 练习题

### 实践题

1. **熟悉界面：** 打开VSCode，依次点击左侧边栏的每个图标，了解它们的功能：
   - 资源管理器（文件图标）
   - 搜索（放大镜图标）
   - 源代码管理（分支图标）
   - 运行和调试（播放图标）
   - 扩展（方块图标）

2. **个性化设置：** 尝试修改以下设置并观察变化：
   - 调整字体大小为16px
   - 更改主题颜色（搜索"color theme"）
   - 开启自动保存功能

3. **快捷键练习：** 创建一个文本文件，练习使用以下快捷键：
   - 输入几行文字后用 `Ctrl + Z` 撤销
   - 用 `Ctrl + F` 查找某个词
   - 用 `Ctrl + H` 替换某个词

### 选择题

1. **打开VSCode扩展面板的快捷键是？**
   - A. Ctrl + E
   - B. Ctrl + Shift + E
   - C. Ctrl + Shift + X ✓
   - D. Ctrl + X

2. **以下哪个不是VSCode的优点？**
   - A. 完全免费
   - B. 启动速度快
   - C. 只能在Windows上使用 ✓
   - D. 支持丰富的扩展

---

## ❓ 常见问题排查

### Q1：安装完成后找不到VSCode在哪里？

**A：** 检查以下几个位置：
- **桌面** - 是否有VSCode快捷方式图标
- **开始菜单** - 搜索 "Visual Studio Code"
- **安装目录** - `C:\Users\你的用户名\AppData\Local\Programs\Microsoft VS Code\Code.exe`
- **右键菜单** - 在任意文件夹上右键，看是否有"用Code打开"

如果都没有，重新运行安装程序，确保勾选了"添加桌面快捷方式"选项。

### Q2：中文语言包安装失败怎么办？

**A：** 可能的原因和解决方法：

1. **网络问题**：检查网络连接，尝试使用代理或镜像
2. **VSCode版本过低**：更新VSCode到最新版本
3. **手动安装**：
   - 访问扩展市场网页：https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans
   - 点击 "Download Extension" 下载 `.vsix` 文件
   - 在VSCode中：扩展面板 → 右上角 `...` → "从VSIX安装"

### Q3：VSCode界面是英文的，怎么改成中文？

**A：** 按照教程第五节的步骤安装中文语言包即可。如果已经安装但还是英文：
1. 按 `Ctrl + Shift + P` 打开命令面板
2. 输入 `Configure Display Language`
3. 选择 `zh-cn` (简体中文)
4. 重启VSCode

### Q4：VSCode打开很慢或卡顿怎么办？

**A：** 优化建议：

1. **禁用不必要的扩展** - 扩展太多会影响性能
2. **排除大文件夹** - 文件监视会消耗资源
   ```json
   "files.watcherExclude": {
     "**/node_modules/**": true,
     "**/.git/**": true
   }
   ```
3. **增加内存限制**（高级）：
   ```json
   "maxMemory": 4096
   ```
4. **检查杀毒软件** - 有时会扫描VSCode进程导致卡顿

### Q5：如何在VSCode中打开文件夹而不是单个文件？

**A：** 有多种方法：

1. **菜单操作**：文件 → 打开文件夹 → 选择项目文件夹
2. **快捷键**：`Ctrl + K` 然后 `Ctrl + O`（注意是连续按）
3. **拖拽**：直接将文件夹拖到VSCode窗口中
4. **命令行**：在文件夹中右键 → "用Code打开"（需要安装时勾选此选项）

### Q6：VSCode和Visual Studio有什么区别？我应该用哪个？

**A：** 简单对比：

| 特性 | VSCode | Visual Studio |
|------|--------|---------------|
| 类型 | 编辑器 | 完整IDE |
| 体积 | ~100MB | 几GB |
| 启动速度 | 秒级 | 分钟级 |
| 价格 | 免费 | Community版免费 |
| 适用场景 | Web开发、轻量级项目 | 大型企业应用、完整调试 |
| 学习曲线 | 平缓 | 较陡峭 |

**对于ASP.NET Core初学者，推荐先使用VSCode！** 它更轻量、上手更快，等熟练后再考虑是否需要完整的Visual Studio。

---

## 🎯 下一步

太棒了！现在你已经有了强大的代码编辑器。接下来我们需要安装一些实用的扩展插件来增强VSCode的功能：

- **[必装扩展清单](./04-必装扩展清单.md)** - 安装C#开发必需的5个扩展

然后就可以正式开始编写第一个ASP.NET Core项目了！

---

## 📚 参考资料

- [VSCode官方文档](https://code.visualstudio.com/docs)
- [VSCode快捷键大全](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)
- [VSCode设置文档](https://code.visualstudio.com/docs/getstarted/settings)
- [中文语言包扩展页面](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans)

---

**⏱️ 预计完成时间：25-35分钟**
**📖 难度等级：★★☆☆☆（简单）**
