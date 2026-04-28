# VSCode 快捷键大全

> 掌握 VS Code 快捷键是提升开发效率的关键。本文整理了 200+ 个常用快捷键，涵盖通用操作、导航、编辑、终端、调试、扩展和窗口管理等各个方面，帮助 ASP.NET 开发者快速上手并精通 VS Code。

---

## 目录

- [一、通用快捷键](#一通用快捷键)
- [二、导航快捷键](#二导航快捷键)
- [三、编辑快捷键](#三编辑快捷键)
- [四、终端快捷键](#四终端快捷键)
- [五、调试快捷键](#五调试快捷键)
- [六、C# 扩展相关快捷键](#六c-扩展相关快捷键)
- [七、窗口管理快捷键](#七窗口管理快捷键)
- [八、自定义快捷键绑定](#八自定义快捷键绑定)
- [九、快捷键学习方法与记忆技巧](#九快捷键学习方法与记忆技巧)
- [十、跨平台快捷键对照表](#十跨平台快捷键对照表)

---

## 一、通用快捷键

通用快捷键是最基础也是最常用的操作，几乎每次编码都会用到。

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 保存文件 | `Ctrl + S` | `Cmd + S` | 保存当前文件 |
| 全部保存 | `Ctrl + K S` | `Cmd + Option + S` | 保存所有打开的文件 |
| 关闭文件 | `Ctrl + W` | `Cmd + W` | 关闭当前标签页 |
| 关闭所有文件 | `Ctrl + K W` | `Cmd + K W` | 关闭所有标签页 |
| 撤销 | `Ctrl + Z` | `Cmd + Z` | 撤销上一步操作 |
| 重做 | `Ctrl + Y` / `Ctrl + Shift + Z` | `Cmd + Shift + Z` / `Cmd + Y` | 重做撤销的操作 |
| 查找 | `Ctrl + F` | `Cmd + F` | 在当前文件中查找 |
| 替换 | `Ctrl + H` | `Cmd + H` | 在当前文件中替换 |
| 全屏模式 | `F11` | `Ctrl + Cmd + F` | 切换全屏显示 |
| 切换侧边栏 | `Ctrl + B` | `Cmd + B` | 显示/隐藏侧边栏 |
| 命令面板 | `Ctrl + Shift + P` | `Cmd + Shift + P` | 打开命令面板（最强大的功能） |
| 打开文件 | `Ctrl + O` | `Cmd + O` | 打开文件选择对话框 |
| 新建文件 | `Ctrl + N` | `Cmd + N` | 创建新文件 |

### 实用技巧

**命令面板 (`Ctrl + Shift + P`) 是 VS Code 的核心入口**，几乎所有功能都可以通过它访问：

```
常用命令示例：
> Toggle Terminal          # 切换终端
> Format Document          # 格式化文档
> Change Language Mode     # 更改语言模式
> Git: Commit              # Git 提交
> C#: Generate Constructors # C# 生成构造函数
```

---

## 二、导航快捷键

高效的代码导航能力能显著减少在大型项目中寻找代码的时间。

### 文件与行导航

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 跳转到指定行 | `Ctrl + G` | `Ctrl + G` | 输入行号跳转 |
| 文件内搜索符号 | `Ctrl + Shift + O` | `Cmd + Shift + O` | 搜索类/方法/属性等符号 |
| 跳转到定义 | `F12` | `F12` | 跳转到符号定义处 |
| 查看定义（预览） | `Alt + F12` | `Option + F12` | 内联预览定义，不离开当前位置 |
| 查看所有引用 | `Shift + F12` | `Shift + F12` | 查看符号的所有引用位置 |
| 返回上一个位置 | `Alt + ←` | `Ctrl + -` | 导航历史后退 |
| 前进到下一个位置 | `Alt + →` | `Ctrl + Shift + -` | 导航历史前进 |
| 打开文件 | `Ctrl + P` | `Cmd + P` | 快速打开文件（模糊匹配） |
| 最近打开的文件 | `Ctrl + E` | `Ctrl + E` | 快速切换最近使用的文件 |

### 符号级导航

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 工作区符号搜索 | `Ctrl + T` | `Cmd + T` | 跨文件搜索符号 |
| 转到实现 | `Ctrl + F12` | `Cmd + F12` | 转到接口的实现 |
| 转到基类型 | `Shift + F12` (上下文) | `Shift + F12` (上下文) | 转到父类/接口 |
| 转到类型定义 | (无默认) | (无默认) | 需要安装扩展支持 |

### 编辑器内导航

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 移动到行首 | `Home` | `Cmd + ←` | 光标移至行首 |
| 移动到行尾 | `End` | `Cmd + →` | 光标移至行尾 |
| 移动到文件开头 | `Ctrl + Home` | `Cmd + ↑` | 光标移至文件开始 |
| 移动到文件末尾 | `Ctrl + End` | `Cmd + ↓` | 光标移至文件结尾 |
| 向上滚动一行 | `Ctrl + Up` | `Ctrl + Up` | 视图上滚，光标不动 |
| 向下滚动一行 | `Ctrl + Down` | `Ctrl + Down` | 视图下滚，光标不动 |
| 跳转到匹配的括号 | `Ctrl + Shift + \` | `Ctrl + Shift + \` | 跳转到对应的括号 |
| 跳转到下一个错误 | `F8` | `F8` | 跳转到下一个错误或警告 |
| 跳转到上一个错误 | `Shift + F8` | `Shift + F8` | 跳转到上一个错误或警告 |
| 折叠当前区域 | `Ctrl + Shift + [` | `Option + Cmd + [` | 折叠代码块 |
| 展开当前区域 | `Ctrl + Shift + ]` | `Option + Cmd + ]` | 展开代码块 |
| 折叠所有区域 | `Ctrl + K Ctrl + 0` | `Cmd + K Cmd + 0` | 折叠所有可折叠区域 |
| 展开所有区域 | `Ctrl + K Ctrl + J` | `Cmd + K Cmd + J` | 展开所有折叠区域 |

### Minimap 导航

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| Minimap 中点击 | 鼠标点击 | 鼠标点击 | 快速定位到对应位置 |
| 缩放 Minimap | `Ctrl + 滚轮` | `Cmd + 滚轮` | 调整 Minimap 大小 |

---

## 三、编辑快捷键

编辑快捷键是日常编码中使用频率最高的，熟练掌握可以大幅提升编码速度。

### 基础编辑

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 复制当前行 | `Ctrl + C` (无选择时) | `Cmd + C` (无选择时) | 无选择内容时复制整行 |
| 剪切当前行 | `Ctrl + X` (无选择时) | `Cmd + X` (无选择时) | 无选择内容时剪切整行 |
| 删除当前行 | `Ctrl + Shift + K` | `Cmd + Shift + K` | 删除整行 |
| 向上移动行 | `Alt + Up` | `Option + Up` | 将当前行上移 |
| 向下移动行 | `Alt + Down` | `Option + Down` | 将当前行下移 |
| 向上复制行 | `Shift + Alt + Up` | `Shift + Option + Up` | 向上复制当前行 |
| 向下复制行 | `Shift + Alt + Down` | `Shift + Option + Down` | 向下复制当前行 |
| 增加缩进 | `Ctrl + ]` | `Cmd + ]` | 增加缩进级别 |
| 减少缩进 | `Ctrl + [` | `Cmd + [` | 减少缩进级别 |
| 注释/取消注释 | `Ctrl + /` | `Cmd + /` | 行注释切换 |
| 块注释 | `Shift + Alt + A` | `Shift + Option + A` | 块注释切换 |
| 自动补全触发 | `Ctrl + Space` | `Ctrl + Space` | 触发 IntelliSense |

### 多光标编辑

多光标编辑是 VS Code 最强大的特性之一，允许同时在多个位置进行编辑。

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 在下方插入光标 | `Ctrl + Alt + Down` | `Cmd + Option + Down` | 在下一行添加光标 |
| 在上方插入光标 | `Ctrl + Alt + Up` | `Cmd + Option + Up` | 在上一行添加光标 |
| 在选中每行末尾添加光标 | `Shift + Alt + I` | `Shift + Option + I` | 批量编辑选中行的末尾 |
| 选择当前词的所有出现 | `Ctrl + Shift + L` | `Cmd + Shift + L` | 同时选择所有相同文本 |
| 选择下一个当前词 | `Ctrl + D` | `Cmd + D` | 逐个选择下一个相同文本 |
| 跳过当前选择 | `Ctrl + K Ctrl + D` | `Cmd + K Cmd + D` | 跳过当前匹配项 |
| 列选择模式 | `Shift + Alt + (鼠标拖拽)` | `Option + (鼠标拖拽)` | 矩形区域选择 |
| 光标位置应用剪贴板 | `Ctrl + Alt + V` | `Cmd + Option + V` | 在每个光标位置粘贴 |

### 文本选择

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 选择当前行 | `Ctrl + L` | `Cmd + L` | 选中整行 |
| 向上扩展选择 | `Shift + Up` | `Shift + Up` | 向上扩展选区 |
| 向下扩展选择 | `Shift + Down` | `Shift + Down` | 向下扩展选区 |
| 选择到行首 | `Shift + Home` | `Shift + Cmd + ←` | 选中到行首 |
| 选择到行尾 | `Shift + End` | `Shift + Cmd + →` | 选中到行尾 |
| 选择到文件开头 | `Ctrl + Shift + Home` | `Shift + Cmd + Up` | 选中到文件开头 |
| 选择到文件末尾 | `Ctrl + Shift + End` | `Shift + Cmd + Down` | 选中到文件末尾 |
| 选择括号内的内容 | `Ctrl + Shift + \` | `Cmd + Shift + \` | 选中匹配括号间的内容 |
| 扩展/收缩选区 | `Shift + Alt + →/←` | `Ctrl + W / Shift + Ctrl + W` | 智能扩展或缩小选择范围 |
| 全选 | `Ctrl + A` | `Cmd + A` | 选中全部内容 |

### 代码格式化与重构

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 格式化文档 | `Shift + Alt + F` | `Shift + Option + F` | 格式化整个文档 |
| 格式化选定部分 | `Ctrl + K Ctrl + F` | `Cmd + K Cmd + F` | 只格式化选中的代码 |
| 转换为大写 | `Ctrl + Shift + U` | `Ctrl + Shift + U` | 选中文本转大写 |
| 转换为小写 | `Ctrl + U` | `Ctrl + U` | 选中文本转小写 |
| 重命名符号 | `F2` | `F2` | 重命名变量/方法（全局重构） |
| 提取方法/变量 | `Ctrl + .` (选择重构选项) | `Cmd + .` (选择重构选项) | 代码重构建议 |
| 导入自动修复 | `Shift + Alt + O` | `Shift + Option + O` | 自动组织 using 语句 |

### ASP.NET 开发者特别关注

对于 C#/ASP.NET 开发，以下编辑快捷键尤为重要：

```csharp
// 使用 Ctrl + . 触发的常见重构：
// 1. 生成构造函数
public class UserService
{
    private readonly IUserRepository _repository;
    // 光标放在 _repository 上按 Ctrl + .
    // -> 生成构造函数参数
}

// 2. 生成接口实现
public class UserRepository : IUserRepository
{
    // 光标放在 IUserRepository 上按 Ctrl + .
    // -> Implement Interface (实现接口)
}

// 3. 提取方法
public IActionResult GetUsers()
{
    var users = _context.Users
        .Where(u => u.IsActive)
        .OrderBy(u => u.Name)
        .ToList();
    // 选中查询逻辑，Ctrl + . -> Extract Method
}
```

---

## 四、终端快捷键

VS Code 内置终端是与开发环境深度集成的，掌握终端快捷键可以无缝切换编码和命令行操作。

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 切换终端显示 | `Ctrl + `` ` `` | `Ctrl + `` ` `` | 显示/隐藏集成终端 |
| 新建终端 | `Ctrl + Shift + `` ` `` | `Cmd + Shift + `` ` `` | 创建新终端实例 |
| 终端分屏 | `Ctrl + Shift + 5` | `Cmd + Shift + 5` | 分割终端窗口 |
| 上一个终端 | `Alt + ↑` | `Cmd + Alt + ↑` | 切换到上一个终端 |
| 下一个终端 | `Alt + ↓` | `Cmd + Alt + ↓` | 切换到下一个终端 |
| 关闭终端 | `Ctrl + Shift + W` | `Cmd + Shift + W` | 关闭当前终端面板 |
| 清空终端 | (需自定义) | (需自定义) | 清除终端输出 |
| 终端向上滚动 | `Shift + PageUp` | `Shift + PageUp` | 向上翻页 |
| 终端向下滚动 | `Shift + PageDown` | `Shift + PageDown` | 向下翻页 |
| 聚焦终端 | `Ctrl + 0` | `Cmd + 0` | 将焦点移到终端 |
| 从终端复制 | `Ctrl + Shift + C` | `Cmd + Shift + C` | 复制终端选区 |
| 粘贴到终端 | `Ctrl + Shift + V` | `Cmd + Shift + V` | 粘贴到终端 |
| 搜索终端输出 | `Ctrl + Shift + F` | `Cmd + Shift + F` | 在终端中搜索 |
| 放大终端字体 | `Ctrl + =` | `Cmd + =` | 放大终端文字 |
| 缩小终端字体 | `Ctrl + -` | `Cmd + -` | 缩小终端文字 |

### 终端实用配置

```json
// settings.json 中的终端配置推荐
{
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "terminal.integrated.fontSize": 14,
  "terminal.integrated.fontFamily": "'Cascadia Code', 'Consolas'",
  "terminal.integrated.cursorBlinking": true,
  "terminal.integrated.scrollback": 10000,
  "terminal.integrated.tabs.enabled": true,
  "terminal.integrated.splitCwd": "initial"
}
```

---

## 五、调试快捷键

调试是开发过程中不可或缺的环节，VS Code 提供了完整的调试体验。

### 核心调试操作

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 开始调试 | `F5` | `F5` | 启动调试会话 |
| 启动但不调试 | `Ctrl + F5` | `Cmd + F5` | 运行不进入调试模式 |
| 切换断点 | `F9` | `F9` | 在当前行设置/取消断点 |
| 单步跳过 | `F10` | `F10` | 执行当前行，不进入函数 |
| 单步进入 | `F11` | `F11` | 进入函数内部执行 |
| 单步跳出 | `Shift + F11` | `Shift + F11` | 跳出当前函数 |
| 重启调试 | `Ctrl + Shift + F5` | `Cmd + Shift + F5` | 重新启动调试 |
| 停止调试 | `Shift + F5` | `Shift + F5` | 停止调试会话 |
| 继续执行 | `F5` | `F5` | 继续运行到下一个断点 |

### 断点管理

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 条件断点 | 右键断点 -> 编辑断点 | 右键断点 -> 编辑断点 | 设置条件表达式 |
| 日志断点 | 右键断点 -> 编辑断点 -> 日志点消息 | 同左 | 输出日志而不暂停 |
| 函数断点 | 断点面板 -> + 号 | 断点面板 -> + 号 | 按函数名设置断点 |
| 启用/禁用所有断点 | (需自定义) | (需自定义) | 批量控制断点 |
| 切换内联断点 | `Shift + F9` | `Shift + F9` | 设置行内断点（Lambda等） |

### 调试视图导航

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 聚焦变量窗口 | (需自定义) | (需自定义) | 查看局部变量 |
| 聚焦监视窗口 | (需自定义) | (需自定义) | 添加监视表达式 |
| 聚焦点调用堆栈 | (需自定义) | (需自定义) | 查看调用链 |
| 聚焦断点面板 | `Ctrl + Shift + F8` | `Cmd + Shift + F8` | 管理所有断点 |
| 切换调试工具栏可见性 | (需自定义) | (需自定义) | 显示/隐藏调试栏 |

### ASP.NET Core 调试场景

```csharp
// 调试 API 控制器时的关键断点设置位置
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    // F9: 在这里设断点检查请求参数
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUser(int id)
    {
        // F11: 进入服务层查看业务逻辑
        var user = await _userService.GetUserByIdAsync(id);

        if (user == null)
            return NotFound(); // F10: 跳过返回语句

        return Ok(user);
    }
}

// 调试中间件时的建议
public class RequestLoggingMiddleware
{
    // 在 InvokeAsync 方法中设断点
    // 可以看到每个请求的完整生命周期
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        _logger.LogInformation(
            "Request {Method} {Path}", context.Request.Method, context.Request.Path);
        await next(context);
    }
}
```

---

## 六、C# 扩展相关快捷键

安装 C# Dev Kit 和相关扩展后，会获得大量 C# 特定的快捷键。

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 快速修复/重构 | `Ctrl + .` | `Cmd + .` | 显示代码操作菜单 |
| 查看所有引用 | `Shift + F12` | `Shift + F12` | 查找所有引用位置 |
| 重命名符号 | `F2` | `F2` | 全局重命名 |
| 生成代码 | `Ctrl + .` -> 选择选项 | `Cmd + .` -> 选择选项 | 生成构造函数/属性/接口实现等 |
| 查看层次结构 | `Shift + Alt + F12` | `Shift + Option + F12` | 类型继承层次 |
| 导航到基类型 | (通过命令面板) | (通过命令面板) | Go to Base |
| 查看文档 | `Ctrl + Shift + Space` | `Ctrl + Shift + Space` | 参数提示和文档 |
| 组织 Using | `Shift + Alt + O` | `Shift + Option + O` | 移除未使用的 using 并排序 |
| 提取接口 | `Ctrl + R Ctrl + I` | `Cmd + R Cmd + I` | 从类提取接口 |
| 预览反编译源码 | `Alt + F12` | `Option + F12` | 查看 DLL 中的源码 |

### C# Dev Kit 特有功能

```
# 通过命令面板访问的 C# 特有功能：
C#: Add Parameter                    # 添加方法参数
C#: Change Signature                 # 修改方法签名
C#: Create Delegate/Event/Field      # 创建委托/事件/字段
C#: Generate Constructor             # 生成构造函数
C#: Generate Equals/GetHashCode      # 生成相等性方法
C#: Implement Interface              # 实现接口
C#: Introduce Field                  # 引入字段
C#: Introduce Local                  # 引入局部变量
C#: Introduce Parameter              # 引入参数
C#: Move Type to File                # 将嵌套类型移到独立文件
C#: Wrap with try-catch              # 用 try-catch 包裹
.NET: Generate New Project           # 创建新项目
.NET: Publish Project                # 发布项目
```

---

## 七、窗口管理快捷键

高效的多窗口和多文件管理能力对于处理大型解决方案至关重要。

### 编辑器分组

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 左右分屏 | `Ctrl + \` | `Cmd + \` | 拆分编辑器为左右两栏 |
| 聚焦左侧编辑器 | `Ctrl + K Ctrl + ←` | `Cmd + K Cmd + ←` | 焦点移到左边编辑器 |
| 聚焦右侧编辑器 | `Ctrl + K Ctrl + →` | `Cmd + K Cmd + →` | 焦点移到右边编辑器 |
| 移动编辑器到左边组 | `Ctrl + K Shift + ←` | `Cmd + K Shift + ←` | 当前文件移到左边组 |
| 移动编辑器到右边组 | `Ctrl + K Shift + →` | `Cmd + K Shift + →` | 当前文件移到右边组 |
| 切换编辑器组大小 | `Ctrl + K Ctrl + =` | `Cmd + K Cmd + =` | 调整分组比例 |
| 合并所有编辑器组 | `Ctrl + K Ctrl + 0` | `Cmd + K Cmd + 0` | 合并为单栏 |

### 标签页操作

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 切换到下一个标签页 | `Ctrl + PageDown` | `Ctrl + PageDown` / `Option + Cmd + →` | 下一个标签 |
| 切换到上一个标签页 | `Ctrl + PageUp` | `Ctrl + PageUp` / `Option + Cmd + ←` | 上一个标签 |
| 打开最近关闭的文件 | `Ctrl + Shift + T` | `Cmd + Shift + T` | 恢复刚关闭的标签 |
| 保持预览模式打开 | `Ctrl + K Enter` | `Cmd + K Enter` | 锁定预览标签不自动切换 |
| 向右拆分编辑器 | `Ctrl + K Ctrl + →` | `Cmd + K Cmd + →` | 向右拆分 |
| 向左拆分编辑器 | `Ctrl + K Ctrl + ←` | `Cmd + K Cmd + ←` | 向左拆开 |

### 侧边栏与面板

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 显示资源管理器 | `Ctrl + Shift + E` | `Cmd + Shift + E` | 打开文件浏览器 |
| 显示搜索 | `Ctrl + Shift + F` | `Cmd + Shift + F` | 全局搜索 |
| 显示源代码管理 | `Ctrl + Shift + G` | `Cmd + Shift + G` | Git 视图 |
| 显示运行和调试 | `Ctrl + Shift + D` | `Cmd + Shift + D` | 调试视图 |
| 显示扩展 | `Ctrl + Shift + X` | `Cmd + Shift + X` | 扩展商店 |
| 显示输出面板 | `Ctrl + Shift + U` | `Cmd + Shift + U` | 输出/问题面板 |
| 显示问题面板 | `Ctrl + Shift + M` | `Cmd + Shift + M` | 错误和警告 |
| 切换面板位置 | `Ctrl + J` | `Cmd + J` | 面板在底部/右侧切换 |
| 缩放重置 | `Ctrl + NumPad0` | `Cmd + NumPad0` | 重置界面缩放 |
| 放大 | `Ctrl + =` / `Ctrl + ++` | `Cmd + =` | 放大界面 |
| 缩小 | `Ctrl + -` | `Cmd + -` | 缩小界面 |

### 多工作区

| 功能 | Windows/Linux | Mac | 说明 |
|------|---------------|-----|------|
| 保存工作区 | (通过文件菜单) | (通过文件菜单) | 保存为 .code-workspace 文件 |
| 打开工作区 | (通过文件菜单) | (通过文件菜单) | 打开已保存的工作区 |
| 添加文件夹到工作区 | (通过文件菜单) | (通过文件菜单) | 多根目录工作区 |

---

## 八、自定义快捷键绑定

VS Code 允许用户完全自定义快捷键，以适应个人习惯或解决冲突。

### 打开快捷键配置

有两种方式打开快捷键配置文件：

1. **图形界面方式**：`Ctrl + K Ctrl + S` 或通过菜单：文件 > 首选项 > 键盘快捷方式
2. **直接编辑 JSON**：在命令面板输入 `Open Keyboard Shortcuts (JSON)`

### keybindings.json 配置示例

以下是一个针对 ASP.NET 开发优化的自定义快捷键配置：

```json
// 将以下内容添加到 keybindings.json
[
  // ===== 自定义快捷键示例 =====

  // 1. 终端相关增强
  {
    "key": "ctrl+shift+k",
    "command": "workbench.action.terminal.clear",
    "when": "terminalFocus"
  },
  {
    "key": "ctrl+alt+t",
    "command": "workbench.action.terminal.toggleTerminal"
  },
  {
    "key": "ctrl+shift+t",
    "command": "workbench.action.terminal.new"
  },

  // 2. 文件操作快捷键
  {
    "key": "ctrl+numpad_add",
    "command": "workbench.files.action.showOpenedFileInNewWindow",
    "when": "!inEditor && !editorAreaVisible"
  },

  // 3. 调试增强
  {
    "key": "f7",
    "command": "workbench.debug.action.focusBreakpointsView"
  },
  {
    "key": "f6",
    "command": "workbench.debug.action.focusWatchView"
  },
  {
    "key": "f4",
    "command": "workbench.debug.action.focusVariablesView"
  },

  // 4. Git 快捷键
  {
    "key": "ctrl+g ctrl+c",
    "command": "git.commit"
  },
  {
    "key": "ctrl+g ctrl+p",
    "command": "git.push"
  },
  {
    "key": "ctrl+g ctrl+l",
    "command": "git.pull"
  },

  // 5. C# 开发专用
  {
    "key": "alt+f7",
    "command": "csharp.showImplementations",
    "when": "editorLangId == 'csharp'"
  },
  {
    "key": "ctrl+shift+r",
    "command": "editor.action.rename.onType",
    "when": "editorHasRenameProvider && editorTextFocus && !editorReadonly"
  }
]
```

### 快捷键语法说明

```json
// keybindings.json 基本格式
[
  {
    "key": "ctrl+shift+p",       // 必填：按键组合
    "command": "workbench.action.showCommands",  // 必填：执行的命令
    "when": "editorTextFocus",   // 可选：生效条件（when 子句）
    "args": {}                   // 可选：传递给命令的参数
  }
]
```

#### 常用按键标识符

| 按键 | 标识符 |
|------|--------|
| Ctrl | `ctrl` |
| Shift | `shift` |
| Alt | `alt` |
| Windows | `win` |
| Command (Mac) | `cmd` |
| 功能键 | `f1` - `f19` |
| 数字小键盘 | `numpad0` - `numpad9`, `numpad_multiply` 等 |

#### When 子句常用条件

```
editorTextFocus          # 编辑器获得焦点
editorHasSelection       # 有文本被选中
editorIsOpen             # 编辑器已打开
editorLangId == 'csharp' # 语言是 C#
terminalFocus            # 终端获得焦点
filesExplorerFocus       # 资源管理器获得焦点
sideBarFocus             # 侧边栏获得焦点
inQuickOpen              # 快速打开对话框激活
listFocus                # 列表控件获得焦点
!editorReadonly          # 编辑器非只读
```

### 解决快捷键冲突

当多个扩展使用相同的快捷键时，可以通过以下方式排查和解决：

```
# 步骤1：在键盘快捷方式设置中搜索冲突的命令
# 步骤2：查看哪些命令使用了相同的快捷键
# 步骤3：右键选择"更改键绑定"或"移除键绑定"

# 或者直接在 keybindings.json 中覆盖：
{
  "key": "ctrl+.",           // 冲突的快捷键
  "command": "-conflictingCommand",  // 加 - 前缀表示禁用原有绑定
  "when": "editorTextFocus"
},
{
  "key": "ctrl+.",
  "command": "myPreferredCommand",   // 绑定到你想要的命令
  "when": "editorTextFocus"
}
```

---

## 九、快捷键学习方法与记忆技巧

掌握大量快捷键需要系统性的学习方法和持续练习。

### 学习路径建议

```
第一阶段（第1周）：基础生存
├── Ctrl+S / Ctrl+Z / Ctrl+F / Ctrl+H     # 保存/撤销/查找/替换
├── Ctrl+P                                  # 快速打开文件
├── Ctrl+Shift+P                            # 命令面板
└── Ctrl+B                                  # 切换侧边栏

第二阶段（第2-3周）：效率提升
├── F5/F9/F10/F11                           # 调试核心四件套
├── F12 / Shift+F12                         # 定义/引用跳转
├── Ctrl+D / Ctrl+L                         # 多光标/选择行
├── Alt+↑↓                                  # 行移动
└── Ctrl+.                                   # 快速修复

第三阶段（第4周及以后）：专家水平
├── 所有导航快捷键
├── 自定义快捷键绑定
├── 高级编辑操作
└── 工作流优化组合
```

### 记忆技巧

**1. 分类记忆法**

将快捷键按功能分类，每天专注一类：

- **周一**：通用 + 文件操作
- **周二**：导航 + 跳转
- **周三**：编辑 + 多光标
- **周四**：终端 + 调试
- **周五**：窗口管理 + 扩展特有

**2. 动作联想记忆法**

根据快捷键的功能联想其含义：

- **F**ind -> `Ctrl+F` 查找
- **S**ave -> `Ctrl+S` 保存
- **Z** 字形回退 -> `Ctrl+Z` 撤销
- **P**ick file -> `Ctrl+P` 选择文件
- **D**uplicate -> `Ctrl+D` 选择重复项
- **L**ine select -> `Ctrl+L` 选择行
- **G**o to line -> `Ctrl+G` 跳转行号

**3. 肌肉记忆训练法**

刻意练习特定快捷键组合：

```bash
# 每天练习计划：
# 1. 打开 VS Code 后，强制自己只用快捷键操作 30 分钟
# 2. 使用 Key Promoter 扩展提醒自己
# 3. 每学会一个新快捷键，就强迫自己使用一周
# 4. 定期复习遗忘的快捷键
```

**4. 场景化记忆法**

结合实际开发场景记忆：

```
场景1：修改 Bug
  Ctrl+P -> 打开文件 -> F12 -> 跳转定义 -> Ctrl+F -> 搜索
  -> F9 -> 设断点 -> F5 -> 调试 -> F10/F11 -> 单步执行

场景2：重构代码
  Ctrl+D -> 选择变量 -> F2 -> 重命名 -> Ctrl+. -> 应用修改

场景3：新建功能
  Ctrl+N -> 新建文件 -> Ctrl+Space -> 智能提示 -> Tab -> 补全
  -> Shift+Alt+F -> 格式化 -> Ctrl+S -> 保存
```

### 推荐的学习辅助工具

| 扩展名称 | 功能描述 |
|----------|----------|
| Key Promoter VSCode | 当你使用鼠标操作时，显示对应的快捷键 |
| Shortcut Menu Bar | 底部状态栏显示常用快捷键按钮 |
| Print Keyboard Shortcuts | 生成快捷键参考卡片 PDF |
| VSCode Great Icons | 配合快捷键提升视觉识别度 |

---

## 十、跨平台快捷键对照表

不同操作系统下的主要差异汇总。

### 主修饰键对照

| 操作 | Windows/Linux | macOS |
|------|---------------|-------|
| 主修饰键 | `Ctrl` | `Cmd (⌘)` |
| 辅助修饰键 | `Alt` | `Option (⌥)` |
| 系统键 | `Win` | `Control (⌃)` |

### 常用快捷键完整对照表

| 功能 | Windows/Linux | macOS | 备注 |
|------|---------------|-------|------|
| 命令面板 | `Ctrl+Shift+P` | `Cmd+Shift+P` | 相同 |
| 快速打开文件 | `Ctrl+P` | `Cmd+P` | 相同 |
| 保存 | `Ctrl+S` | `Cmd+S` | 相同 |
| 全部保存 | `Ctrl+K S` | `Cmd+Option+S` | 不同 |
| 关闭标签 | `Ctrl+W` | `Cmd+W` | 相同 |
| 撤销 | `Ctrl+Z` | `Cmd+Z` | 相同 |
| 重做 | `Ctrl+Y` | `Cmd+Shift+Z` | 不同 |
| 查找 | `Ctrl+F` | `Cmd+F` | 相同 |
| 替换 | `Ctrl+H` | `Cmd+H` | 相同 |
| 行操作 | | | |
| 删除行 | `Ctrl+Shift+K` | `Cmd+Shift+K` | 相同 |
| 复制行 | `Ctrl+C` (无选择) | `Cmd+C` (无选择) | 相同 |
| 上移行 | `Alt+Up` | `Option+Up` | Alt→Option |
| 下移行 | `Alt+Down` | `Option+Down` | Alt→Option |
| 注释 | `Ctrl+/` | `Cmd+/` | 相同 |
| 导航 | | | |
| 行跳转 | `Ctrl+G` | `Ctrl+G` | 相同 |
| 符号搜索 | `Ctrl+Shift+O` | `Cmd+Shift+O` | 相同 |
| 定义跳转 | `F12` | `F12` | 相同 |
| 引用查找 | `Shift+F12` | `Shift+F12` | 相同 |
| 文件头 | `Ctrl+Home` | `Cmd+Up` | 不同 |
| 文件尾 | `Ctrl+End` | `Cmd+Down` | 不同 |
| 多光标 | | | |
| 下方光标 | `Ctrl+Alt+Down` | `Cmd+Option+Down` | 组合不同 |
| 上方光标 | `Ctrl+Alt+Up` | `Cmd+Option+Up` | 组合不同 |
| 选择所有匹配 | `Ctrl+Shift+L` | `Cmd+Shift+L` | 相同 |
| 逐个选择 | `Ctrl+D` | `Cmd+D` | 相同 |
| 编辑 | | | |
| 格式化 | `Shift+Alt+F` | `Shift+Option+F` | Alt→Option |
| 重命名 | `F2` | `F2` | 相同 |
| 快速修复 | `Ctrl+.` | `Cmd+.` | 相同 |
| 终端 | | | |
| 切换终端 | `Ctrl+\`` | `Ctrl+\`` | 相同 |
| 新建终端 | `Ctrl+Shift+\`` | `Cmd+Shift+\`` | 相同 |
| 调试 | | | |
| 开始/继续 | `F5` | `F5` | 相同 |
| 断点 | `F9` | `F9` | 相同 |
| 单步跳过 | `F10` | `F10` | 相同 |
| 单步进入 | `F11` | `F11` | 相同 |
| 跳出 | `Shift+F11` | `Shift+F11` | 相同 |
| 窗口 | | | |
| 分屏 | `Ctrl+\`` | `Cmd+\`` | 相同 |
| 侧边栏 | `Ctrl+B` | `Cmd+B` | 相同 |
| 全屏 | `F11` | `Ctrl+Cmd+F` | 不同 |

### 平台适配建议

如果你需要在多个平台上工作，建议采用以下策略：

1. **优先使用跨平台一致的快捷键**：如 `F5/F9/F10/F11`、`Ctrl+P`、`F2` 等
2. **为平台差异较大的操作建立肌肉记忆**：如 `Alt+Up` vs `Option+Up`
3. **使用 Settings Sync 同步配置**：但注意快捷键可能需要手动调整
4. **考虑使用统一的键盘映射方案**：在 Mac 上安装 Karabiner 等工具模拟 Windows 快捷键布局

---

## 附录：速查卡打印版

### 最常用的 20 个快捷键（必须记住）

```
┌─────────────────────────────────────────────────────┐
│  VS Code 核心快捷键 TOP 20                          │
├─────────┬───────────────────────────────────────────┤
│ Ctrl+P  │ 快速打开文件                               │
│ Ctrl+S  │ 保存文件                                   │
│ Ctrl+Z  │ 撤销                                       │
│ Ctrl+/  │ 行注释                                     │
│ Ctrl+D  │ 选择下一个匹配                              │
│ Ctrl+L  │ 选择整行                                   │
│ Ctrl+G  │ 跳转到指定行                               │
│ Ctrl+F  │ 查找                                       │
│ Ctrl+H  │ 替换                                       │
│ Ctrl+B  │ 切换侧边栏                                 │
│ Ctrl+`  │ 切换终端                                   │
│ Ctrl+.  │ 快速修复/重构                              │
│ F2      │ 重命名                                     │
│ F5      │ 开始调试                                   │
│ F9      │ 切换断点                                   │
│ F10     │ 单步跳过                                   │
│ F11     │ 单步进入                                   │
│ F12     │ 跳转到定义                                 │
│ Shift+F12│ 查看所有引用                              │
│ Ctrl+Shift+P│ 命令面板                               │
└─────────┴───────────────────────────────────────────┘
```

### ASP.NET 开发者专属 TOP 10

```
┌─────────────────────────────────────────────────────┐
│  ASP.NET Core 开发必备快捷键                        │
├──────────────┬──────────────────────────────────────┤
│ Ctrl+.       │ C# 快速修复/重构                      │
│ F12          │ 跳转到定义                            │
│ Shift+F12    │ 查看所有引用                          │
│ F2           │ 重命名符号                            │
│ Shift+Alt+O  │ 整理 Using 语句                       │
│ Shift+Alt+F  │ 格式化文档                            │
│ F5/F9-F11    │ 调试核心操作                          │
│ Ctrl+P       │ 快速打开文件                          │
│ Ctrl+Shift+F │ 全局搜索                             │
│ Ctrl+`       │ 切换终端（运行 dotnet 命令）         │
└──────────────┴──────────────────────────────────────┘
```

---

## 总结

本文整理了超过 200 个 VS Code 快捷键，涵盖了从基础操作到高级功能的方方面面。作为 ASP.NET 开发者，建议按照以下优先级学习和使用这些快捷键：

1. **立即掌握**：`Ctrl+P`、`Ctrl+Shift+P`、`Ctrl+.`、`F5/F9/F10/F11`、`F12`
2. **本周掌握**：多光标编辑、行操作、格式化、导航跳转
3. **持续优化**：自定义快捷键、工作区配置、扩展特定快捷键

记住：快捷键的价值不在于数量，而在于熟练程度。每天坚持使用，一个月后你会发现自己的开发效率有显著提升。
