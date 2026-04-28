# LINQ基础

> **学习目标**：掌握LINQ查询语法和常用方法，能够用优雅的方式对集合进行筛选、排序、投影等操作
>
> **前置知识**：面向对象基础（建议先学习前三节）
>
> **预计用时**：40-55分钟

---

## 一、为什么需要LINQ？

### 1.1 没有LINQ时的痛苦

想象你在开发一个电商后台，需要从1000个订单中找出：
- 今天创建的订单
- 金额大于1000元的
- 按金额降序排列
- 只取前10条
- 只要订单号和金额两个字段

**传统方式（又长又难读）：**

```csharp
// 传统for循环方式 - 冗长且容易出错
List<Order> todayOrders = new List<Order>();
DateTime today = DateTime.Today;

foreach (var order in allOrders)
{
    if (order.CreateTime.Date == today && order.Amount > 1000)
    {
        todayOrders.Add(order);
    }
}

// 手动排序（冒泡排序？快排？写起来很麻烦）
todayOrders.Sort((a, b) => b.Amount.CompareTo(a.Amount));

// 手动取前10条
List<Order> top10 = new List<Order>();
for (int i = 0; i < Math.Min(10, todayOrders.Count); i++)
{
    top10.Add(todayOrders[i]);
}

// 再手动提取需要的字段...
// 代码已经几十行了，而且很难维护！
```

**使用LINQ后（一行搞定）：**

```csharp
// LINQ方式 - 简洁、可读性强
var result = allOrders
    .Where(o => o.CreateTime.Date == DateTime.Today)  // 过滤：今天的订单
    .Where(o => o.Amount > 1000)                       // 过滤：金额>1000
    .OrderByDescending(o => o.Amount)                  // 排序：按金额降序
    .Take(10)                                          // 取前10条
    .Select(o => new { o.OrderNo, o.Amount });         // 投影：只要这两个字段

// 就这么简单！代码即文档，一目了然
```

### 1.2 LINQ的核心价值

| 特性 | 说明 | 类比 |
|------|------|------|
| **统一查询接口** | 无论数据源是数组、列表、数据库还是XML，都用相同的语法 | 万能钥匙能开所有门 |
| **声明式编程** | 告诉计算机"要什么"，而不是"怎么做" | 点菜时说"我要宫保鸡丁"，而不是教厨师怎么做 |
| **类型安全** | 编译时检查错误，而不是运行时才发现 | 拼写检查器在你写完时就标错 |
| **可组合性** | 多个操作可以像搭积木一样串联 | 自来水管，一节节接起来 |

### 1.3 数据库查询类比（重要！）

如果你学过SQL，那么LINQ会非常亲切：

| SQL | LINQ方法 | 说明 |
|-----|----------|------|
| `WHERE` | `.Where()` | 过滤/筛选 |
| `SELECT` | `.Select()` | 投影/选择列 |
| `ORDER BY` | `.OrderBy()` / `.OrderByDescending()` | 排序 |
| `TOP / LIMIT` | `.Take()` | 取前N条 |
| `COUNT(*)` | `.Count()` | 计数 |
| `SUM()` | `.Sum()` | 求和 |
| `AVG()` | `.Average()` | 平均值 |
| `MAX/MIN` | `.Max()` / `.Min()` | 最大/最小值 |
| `DISTINCT` | `.Distinct()` | 去重 |
| `GROUP BY` | `.GroupBy()` | 分组 |
| `HAVING` | `.Where()` (在GroupBy之后) | 分组后过滤 |
| `JOIN` | `.Join()` | 连接表 |

---

## 二、Lambda表达式初步

在深入学习LINQ之前，必须先理解它的好搭档——**Lambda表达式**。

### 2.1 什么是Lambda表达式？

Lambda表达式是一种简洁的**匿名函数**写法。

**演进过程：**

```csharp
// 方式1：普通命名方法（最传统）
public bool IsExpensive(Order order)
{
    return order.Amount > 1000;
}
var result = orders.Where(IsExpensive);

// 方式2：匿名委托（C# 2.0）
var result = orders.Where(delegate(Order order) {
    return order.Amount > 1000;
});

// 方式3：Lambda表达式（C# 3.0+）- 推荐！
var result = orders.Where(order => order.Amount > 1000);
//                      ↑        ↑              ↑
//                    参数     =>运算符       方法体

// 更简洁的写法（参数类型自动推断）
var result = orders.Where(o => o.Amount > 1000);  // o就是order的缩写
```

### 2.2 Lambda表达式的基本语法

```csharp
// 基本结构: (参数) => { 表达式或语句块 }

// 1. 无参数
() => Console.WriteLine("Hello");

// 2. 单个参数（括号可以省略）
x => x * x;                          // 返回x的平方
name => name.ToUpper();             // 转大写
user => user.Age >= 18;              // 判断是否成年

// 3. 多个参数（括号不能省略）
(x, y) => x + y;                     // 两数相加
(first, second) => first.CompareTo(second);  // 比较

// 4. 多行语句块（需要花括号）
(x, y) =>
{
    int sum = x + y;
    int product = x * y;
    return sum + product;
};

// 5. 隐式类型推断（编译器自动识别类型）
numbers.Where(n => n > 5);           // n自动推断为int
users.Where(u => u.IsActive);        // u自动推断为User类型
```

### 2.3 实际应用示例

```csharp
public class LambdaDemo
{
    public void DemonstrateLambdas()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        List<string> names = new List<string> { "Alice", "Bob", "Charlie", "David", "Eve" };

        // 示例1：筛选偶数
        var evens = numbers.Where(n => n % 2 == 0);
        Console.WriteLine($"偶数: [{string.Join(", ", evens)}]");
        // 输出: [2, 4, 6, 8, 10]

        // 示例2：筛选长度大于3的名字
        var longNames = names.Where(name => name.Length > 3);
        Console.WriteLine($"长名字: [{string.Join(", ", longNames)}]");
        // 输出: [Alice, Charlie, David]

        // 示例3：转换为大写
        var upperNames = names.Select(name => name.ToUpper());
        Console.WriteLine($"大写: [{string.Join(", ", upperNames)}]");
        // 输出: [ALICE, BOB, CHARLIE, DAVID, EVE]

        // 示例4：计算平方
        var squares = numbers.Select(n => n * n);
        Console.WriteLine($"平方: [{string.Join(", ", squares)}]");
        // 输出: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

        // 示例5：复杂条件组合
        var result = numbers
            .Where(n => n > 3)            // 大于3
            .Where(n => n < 9)            // 小于9
            .Select(n => n * 10);         // 乘以10

        Console.WriteLine($"3<n<9 且 ×10: [{string.Join(", ", result)}]");
        // 输出: [40, 50, 60, 70, 80]
    }
}

// 运行演示
var demo = new LambdaDemo();
demo.DemonstrateLambdas();
```

---

## 三、常用LINQ方法详解

为了演示方便，我们先定义一些测试数据：

```csharp
// ===== 测试数据定义 =====
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    public int Stock { get; set; }
    public bool IsOnSale { get; set; }
    public DateTime CreateDate { get; set; }
    public double Rating { get; set; }

    public override string ToString() => $"{Name} (¥{Price:F2})";
}

public class Order
{
    public int OrderId { get; set; }
    public string CustomerName { get; set; }
    public decimal Amount { get; set; }
    public DateTime OrderDate { get; set; }
    public string Status { get; set; }
    public string City { get; set; }
}

public class TestData
{
    public static List<Product> GetProducts()
    {
        return new List<Product>
        {
            new Product { Id = 1, Name = "iPhone 15 Pro", Price = 8999m,
                        Category = "手机", Stock = 100, IsOnSale = true,
                        CreateDate = new DateTime(2024, 1, 15), Rating = 4.8 },

            new Product { Id = 2, Name = "MacBook Pro 14", Price = 14999m,
                        Category = "电脑", Stock = 50, IsOnSale = false,
                        CreateDate = new DateTime(2024, 1, 20), Rating = 4.9 },

            new Product { Id = 3, Name = "AirPods Pro 2", Price = 1899m,
                        Category = "耳机", Stock = 200, IsOnSale = true,
                        CreateDate = new DateTime(2024, 2, 1), Rating = 4.7 },

            new Product { Id = 4, Name = "iPad Air M2", Price = 4799m,
                        Category = "平板", Stock = 80, IsOnSale = false,
                        CreateDate = new DateTime(2024, 2, 10), Rating = 4.6 },

            new Product { Id = 5, Name = "Apple Watch Ultra 2", Price = 6499m,
                        Category = "手表", Stock = 30, IsOnSale = true,
                        CreateDate = new DateTime(2024, 2, 15), Rating = 4.8 },

            new Product { Id = 6, Name = "Sony WH-1000XM5", Price = 2499m,
                        Category = "耳机", Stock = 150, IsOnSale = true,
                        CreateDate = new DateTime(2024, 3, 1), Rating = 4.7 },

            new Product { Id = 7, Name = "Samsung Galaxy S24", Price = 6999m,
                        Category = "手机", Stock = 120, IsOnSale = false,
                        CreateDate = new DateTime(2024, 3, 5), Rating = 4.5 },

            new Product { Id = 8, Name = "Dell XPS 15", Price = 12999m,
                        Category = "电脑", Stock = 40, IsOnSale = true,
                        CreateDate = new DateTime(2024, 3, 10), Rating = 4.6 },

            new Product { Id = 9, Name = "Kindle Paperwhite", Price = 999m,
                        Category = "电子书", Stock = 300, IsOnSale = false,
                        CreateDate = new DateTime(2024, 3, 15), Rating = 4.4 },

            new Product { Id = 10, Name = "Nintendo Switch OLED", Price = 2399m,
                        Category = "游戏机", Stock = 60, IsOnSale = true,
                        CreateDate = new DateTime(2024, 3, 20), Rating = 4.8 }
        };
    }

    public static List<Order> GetOrders()
    {
        return new List<Order>
        {
            new Order { OrderId = 1001, CustomerName = "张三", Amount = 8999m,
                      OrderDate = new DateTime(2024, 3, 1), Status = "已完成", City = "北京" },
            new Order { OrderId = 1002, CustomerName = "李四", Amount = 14999m,
                      OrderDate = new DateTime(2024, 3, 2), Status = "配送中", City = "上海" },
            new Order { OrderId = 1003, CustomerName = "王五", Amount = 1899m,
                      OrderDate = new DateTime(2024, 3, 3), Status = "已完成", City = "广州" },
            new Order { OrderId = 1004, CustomerName = "赵六", Amount = 6499m,
                      OrderDate = new DateTime(2024, 3, 5), Status = "待付款", City = "深圳" },
            new Order { OrderId = 1005, CustomerName = "钱七", Amount = 12999m,
                      OrderDate = new DateTime(2024, 3, 8), Status = "已取消", City = "杭州" },
            new Order { OrderId = 1006, CustomerName = "张三", Amount = 2499m,
                      OrderDate = new DateTime(2024, 3, 10), Status = "已完成", City = "北京" },
            new Order { OrderId = 1007, CustomerName = "孙八", Amount = 4799m,
                      OrderDate = new DateTime(2024, 3, 12), Status = "配送中", City = "成都" },
            new Order { OrderId = 1008, CustomerName = "周九", Amount = 999m,
                      OrderDate = new DateTime(2024, 3, 15), Status = "已完成", City = "武汉" },
            new Order { OrderId = 1009, CustomerName = "吴十", Amount = 6999m,
                      OrderDate = new DateTime(2024, 3, 18), Status = "待付款", City = "南京" },
            new Order { OrderId = 1010, CustomerName = "李四", Amount = 2399m,
                      OrderDate = new DateTime(2024, 3, 20), Status = "已完成", City = "上海" }
        };
    }
}
```

### 3.1 Where - 筛选过滤（最重要的方法）

`.Where()` 用于从集合中筛选出满足条件的元素，相当于SQL的 `WHERE` 子句。

#### 示例1：基本筛选

```csharp
public class WhereExamples
{
    public void DemonstrateWhere()
    {
        var products = TestData.GetProducts();

        Console.WriteLine("=== Where 筛选示例 ===\n");

        // 场景1：筛选价格大于5000的商品
        var expensiveProducts = products.Where(p => p.Price > 5000);
        Console.WriteLine("【价格>5000的商品】");
        foreach (var p in expensiveProducts)
            Console.WriteLine($"  {p.Name}: ¥{p.Price:N2}");

        // 场景2：筛选正在促销的商品
        var onSaleProducts = products.Where(p => p.IsOnSale);
        Console.WriteLine("\n【促销中的商品】");
        foreach (var p in onSaleProducts)
            Console.WriteLine($"  🏷️  {p.Name}: ¥{p.Price:N2}");

        // 场景3：筛选库存不足的商品（<50）
        var lowStock = products.Where(p => p.Stock < 50);
        Console.WriteLine("\n【库存预警 (<50件)】");
        foreach (var p in lowStock)
            Console.WriteLine($"  ⚠️  {p.Name}: 仅剩{p.Stock}件");

        // 场景4：多条件组合（AND关系）
        var filtered = products.Where(p =>
            p.Category == "手机" &&      // 是手机
            p.IsOnSale &&               // 正在促销
            p.Rating >= 4.5);          // 评分>=4.5

        Console.WriteLine("\n【手机类+促销中+评分≥4.5】");
        foreach (var p in filtered)
            Console.WriteLine($"  ✓ {p.Name} ★{p.Rating} ¥{p.Price:N2}");

        // 场景5：OR条件（满足任一即可）
        var specialProducts = products.Where(p =>
            p.Price > 10000 ||           // 价格过万
            p.Rating >= 4.9);          // 或评分极高

        Console.WriteLine("\n【高价商品或高评分商品】");
        foreach (var p in specialProducts)
            Console.WriteLine($"  ⭐ {p.Name} ¥{p.Price:N2} ★{p.Rating}");

        // 场景6：日期范围筛选
        var recentProducts = products.Where(p =>
            p.CreateDate >= new DateTime(2024, 3, 1) &&
            p.CreateDate <= new DateTime(2024, 3, 31));

        Console.WriteLine($"\n【3月份新增商品 ({recentProducts.Count()}个)】");
        foreach (var p in recentProducts)
            Console.WriteLine($"  {p.Name} - {p.CreateDate:MM-dd}");
    }
}

// 运行示例
var whereDemo = new WhereExamples();
whereDemo.DemonstrateWhere();
```

#### 示例2：订单筛选实战

```csharp
public class OrderFilterDemo
{
    public void FilterOrders()
    {
        var orders = TestData.GetOrders();

        Console.WriteLine("=== 订单筛选实战 ===\n");

        // 任务1：查找所有已完成的订单
        var completedOrders = orders.Where(o => o.Status == "已完成");
        Console.WriteLine($"【已完成订单】共 {completedOrders.Count()} 笔:");
        foreach (var o in completedOrders)
            Console.WriteLine($"  #{o.OrderId} {o.CustomerName,-6} ¥{o.Amount,8:N2} {o.OrderDate:MM-dd}");

        // 任务2：查找大额订单（>5000元）且状态为已完成或配送中
        var largeActiveOrders = orders.Where(o =>
            o.Amount > 5000 &&
            (o.Status == "已完成" || o.Status == "配送中"));

        Console.WriteLine($"\n【大额活跃订单 (>¥5000)】共 {largeActiveOrders.Count()} 笔:");
        foreach (var o in largeActiveOrders)
            Console.WriteLine($"  #{o.OrderId} {o.CustomerName,-6} ¥{o.Amount,8:N2} [{o.Status}]");

        // 任务3：查找特定城市的订单
        string targetCity = "北京";
        var cityOrders = orders.Where(o => o.City == targetCity);
        Console.WriteLine($"\n【{targetCity}的订单】共 {cityOrders.Count()} 笔:");

        // 任务4：查找重复购买的客户（同一客户有多个订单）
        var repeatCustomers = orders
            .GroupBy(o => o.CustomerName)
            .Where(g => g.Count() > 1)
            .Select(g => new { Customer = g.Key, OrderCount = g.Count(), TotalSpent = g.Sum(o => o.Amount) });

        Console.WriteLine($"\n【复购客户】");
        foreach (var c in repeatCustomers)
            Console.WriteLine($"  {c.Customer,-6} 下单{c.OrderCount}次，累计消费 ¥{c.TotalSpent:N2}");

        // 任务5：查找异常订单（金额为0或负数，或者未来日期）
        var abnormalOrders = orders.Where(o =>
            o.Amount <= 0 || o.OrderDate > DateTime.Now);

        if (abnormalOrders.Any())
        {
            Console.WriteLine($"\n⚠️ 【异常订单】共 {abnormalOrders.Count()} 笔需要人工审核!");
        }
        else
        {
            Console.WriteLine("\n✓ 未发现异常订单");
        }
    }
}

// 运行订单筛选
var orderFilter = new OrderFilterDemo();
orderFilter.FilterOrders();
```

### 3.2 Select - 投影转换

`.Select()` 用于将每个元素转换为新的形式，相当于SQL的 `SELECT` 子句。

#### 示例3：各种投影操作

```csharp
public class SelectExamples
{
    public void DemonstrateSelect()
    {
        var products = TestData.GetProducts();

        Console.WriteLine("=== Select 投影示例 ===\n");

        // 场景1：只提取商品名称
        var names = products.Select(p => p.Name);
        Console.WriteLine("【所有商品名称】");
        Console.WriteLine($"  [{string.Join(", ", names)}]");

        // 场景2：提取名称和价格（匿名类型）
        var nameAndPrice = products.Select(p => new { p.Name, p.Price });
        Console.WriteLine("\n【名称和价格】");
        foreach (var item in nameAndPrice)
            Console.WriteLine($"  {item.Name,-22} ¥{item.Price,8:N2}");

        // 场景3：计算并生成新字段
        var productInfo = products.Select(p => new
        {
            p.Name,
            p.Price,
            PriceWithTax = p.Price * 1.13m,   // 含税价
            DiscountedPrice = p.Price * 0.9m, // 打9折后的价格
            CategoryLevel = p.Category == "电脑" ? "高端" :
                           p.Category == "手机" ? "主流" : "其他"
        });

        Console.WriteLine("\n【商品详细信息】");
        foreach (var item in productInfo.Take(5))  // 只显示前5个
        {
            Console.WriteLine($"  {item.Name,-22}");
            Console.WriteLine($"    原价: ¥{item.Price:N2} | 含税: ¥{item.PriceWithTax:N2} | 折后: ¥{item.DiscountedPrice:N2}");
            Console.WriteLine($"    分类级别: {item.CategoryLevel}\n");
        }

        // 场景4：提取索引（重载版本）
        var numberedProducts = products.Select((p, index) => new
        {
            Index = index + 1,
            p.Name,
            p.Category
        });

        Console.WriteLine("【带编号的商品列表】");
        foreach (var item in numberedProducts)
            Console.WriteLine($"  {item.Index,2}. [{item.Category,-6}] {item.Name}");

        // 场景5：嵌套投影（先分组再投影）
        var categorySummary = products
            .GroupBy(p => p.Category)
            .Select(g => new
            {
                Category = g.Key,
                Count = g.Count(),
                AvgPrice = g.Average(p => p.Price),
                TotalStock = g.Sum(p => p.Stock)
            });

        Console.WriteLine("\n【分类统计汇总】");
        foreach (var item in categorySummary.OrderBy(c => c.Category))
        {
            Console.WriteLine($"  {item.Category,-6} | 数量:{item.Count,2} | " +
                            $"均价:¥{item.AvgPrice,8:N0} | 总库存:{item.TotalStock,4}");
        }
    }
}

// 运行投影示例
var selectDemo = new SelectExamples();
selectDemo.DemonstrateSelect();
```

### 3.3 OrderBy / OrderByDescending - 排序

用于对集合进行升序或降序排列。

#### 示例4：多种排序场景

```csharp
public class OrderingExamples
{
    public void DemonstrateOrdering()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("=== 排序示例 ===\n");

        // 场景1：按价格升序（便宜到贵）
        var byPriceAsc = products.OrderBy(p => p.Price);
        Console.WriteLine("【按价格升序】");
        foreach (var p in byPriceAsc)
            Console.WriteLine($"  {p.Name,-22} ¥{p.Price,8:N2}");

        // 场景2：按价格降序（贵到便宜）
        var byPriceDesc = products.OrderByDescending(p => p.Price);
        Console.WriteLine("\n【按价格降序】Top5:");
        foreach (var p byPriceDesc.Take(5))
            Console.WriteLine($"  💰 {p.Name,-22} ¥{p.Price,8:N2}");

        // 场景3：多级排序（先按分类，再按价格降序）
        var multiSort = products
            .OrderBy(p => p.Category)          // 主排序：分类升序
            .ThenByDescending(p => p.Price);   // 次排序：价格降序

        Console.WriteLine("\n【按分类+价格降序】");
        string currentCategory = "";
        foreach (var p in multiSort)
        {
            if (p.Category != currentCategory)
            {
                currentCategory = p.Category;
                Console.WriteLine($"\n  ▶ {currentCategory}:");
            }
            Console.WriteLine($"    {p.Name,-18} ¥{p.Price,8:N2}");
        }

        // 场景4：按字符串长度排序
        var names = products.Select(p => p.Name).OrderBy(n => n.Length);
        Console.WriteLine("\n【按名称长度排序】");
        foreach (var name in names)
            Console.WriteLine($"  {name} ({name.Length}字符)");

        // 场景5：按日期排序订单
        var ordersByDate = orders.OrderByDescending(o => o.OrderDate);
        Console.WriteLine("\n【最新订单 Top5】");
        foreach (var o in ordersByDate.Take(5))
            Console.WriteLine($"  {o.OrderDate:MM-dd} #{o.OrderId} {o.CustomerName,-6} ¥{o.Amount,8:N2} [{o.Status}]");

        // 场景6：自定义排序规则（按库存状态：缺货→低库存→正常）
        var customOrder = products.OrderBy(p =>
        {
            if (p.Stock == 0) return 0;      // 缺货排最前（预警）
            if (p.Stock < 50) return 1;      // 低库存其次
            return 2;                         // 正常最后
        }).ThenBy(p => p.Stock);             // 同级别内按库存数量排序

        Console.WriteLine("\n【按库存紧急程度排序】");
        foreach (var p in customOrder)
        {
            string status = p.Stock == 0 ? "🔴 缺货" :
                           p.Stock < 50 ? "🟡 低库存" : "🟢 正常";
            Console.WriteLine($"  {status} {p.Name,-18} 库存:{p.Stock,4}");
        }
    }
}

// 运行排序示例
var orderingDemo = new OrderingExamples();
orderingDemo.DemonstrateOrdering();
```

### 3.4 First / FirstOrDefault - 获取第一个元素

#### 示例5：获取单个元素的各种方式

```csharp
public class FirstExamples
{
    public void DemonstrateFirstMethods()
    {
        var products = TestData.GetProducts();

        Console.WriteLine("=== First/FirstOrDefault 示例 ===\n");

        // First：返回第一个匹配元素，如果没有则抛异常
        try
        {
            var firstProduct = products.First();  // 集合的第一个元素
            Console.WriteLine($"第一个产品: {firstProduct.Name}");

            var firstExpensive = products.First(p => p.Price > 10000);
            Console.WriteLine($"第一个过万的产品: {firstExpensive.Name} ¥{firstExpensive.Price:N2}");

            // 危险操作！如果找不到会抛异常
            // var notExist = products.First(p => p.Price > 99999);  // 会抛InvalidOperationException!
        }
        catch (Exception ex)
        {
            Console.WriteLine($"❌ 错误: {ex.Message}");
        }

        // FirstOrDefault：更安全，找不到返回默认值
        var firstOrDefault = products.FirstOrDefault();  // 第一个或null
        Console.WriteLine($"\nFirstOrDefault: {firstOrDefault?.Name ?? "空"}");

        var expensiveOrDefault = products.FirstOrDefault(p => p.Price > 10000);
        if (expensiveOrDefault != null)
        {
            Console.WriteLine($"找到过万产品: {expensiveOrDefault.Name}");
        }

        var notFound = products.FirstOrDefault(p => p.Price > 99999);
        Console.WriteLine($"找不存在的: {(notFound == null ? "返回null（安全）" : "找到")}");

        // 实际应用场景：根据ID查找
        int targetId = 5;
        var productById = products.FirstOrDefault(p => p.Id == targetId);
        if (productById != null)
        {
            Console.WriteLine($"\nID={targetId} 的产品: {productById.Name}");
        }
        else
        {
            Console.WriteLine($"\n未找到 ID={targetId} 的产品");
        }

        // Single/SingleOrDefault：确保只有一个匹配项
        var uniqueProduct = products.SingleOrDefault(p => p.Id == 3);
        Console.WriteLine($"\nSingle: {uniqueProduct?.Name}");

        // Last/LastOrDefault：最后一个匹配元素
        var lastOnSale = products.LastOrDefault(p => p.IsOnSale);
        Console.WriteLine($"\n最后一个促销商品: {lastOnSale?.Name}");
    }
}

// 运行First示例
var firstDemo = new FirstExamples();
firstDemo.DemonstrateFirstMethods();
```

### 3.5 Any / All / Contains - 判断是否存在

这些方法返回布尔值，用于判断集合是否满足某个条件。

#### 示例6：存在性判断

```csharp
public class QuantifierExamples
{
    public void DemonstrateQuantifiers()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("=== Any/All/Contains 示例 ===\n");

        // Any：是否存在至少一个满足条件的元素
        bool hasExpensive = products.Any(p => p.Price > 10000);
        Console.WriteLine($"是否有超过1万的商品? {hasExpensive}");

        bool hasOutOfStock = products.Any(p => p.Stock == 0);
        Console.WriteLine($"是否有缺货商品? {hasOutOfStock}");

        bool hasPhone = products.Any(p => p.Category == "手机");
        Console.WriteLine($"是否有手机类商品? {hasPhone}");

        // All：是否所有元素都满足条件
        bool allInStock = products.All(p => p.Stock > 0);
        Console.WriteLine($"\n所有商品都有库存? {allInStock}");

        bool allHighRated = products.All(p => p.Rating >= 4.0);
        Console.WriteLine($"所有商品评分都>=4.0? {allHighRated}");

        bool allOrdersValid = orders.All(o => o.Amount > 0 && o.OrderDate <= DateTime.Now);
        Console.WriteLine($"所有订单都有效? {allOrdersValid}");

        // Contains：是否包含特定值
        List<string> categories = products.Select(p => p.Category).Distinct().ToList();
        bool hasTablet = categories.Contains("平板");
        Console.WriteLine($"\n分类列表包含'平板'? {hasTablet}");

        // 实际业务场景验证
        Console.WriteLine("\n--- 业务验证 ---");

        // 验证1：能否执行某操作
        if (products.Any(p => p.Stock < 10))
        {
            Console.WriteLine("⚠️  警告：部分商品库存严重不足(<10)，请及时补货！");
        }

        // 验证2：批量操作前的安全检查
        if (orders.All(o => o.Status != "配送中"))
        {
            Console.WriteLine("✓ 可以执行系统维护（没有正在配送的订单）");
        }
        else
        {
            Console.WriteLine("⚠️  有订单正在配送中，暂不能执行系统维护");
        }

        // 验证3：用户权限检查
        List<string> userRoles = new List<string> { "User", "VIP" };
        bool canAccessAdmin = userRoles.Any(r => r == "Admin" || r == "SuperAdmin");
        Console.WriteLine($"用户有权访问管理后台? {canAccessAdmin}");

        // 验证4：表单输入验证
        string inputEmail = "test@example.com";
        char[] allowedChars = "abcdefghijklmnopqrstuvwxyz@.-_".ToCharArray();
        bool isValidEmail = inputEmail.All(c => allowedChars.Contains(char.ToLower(c)));
        Console.WriteLine($"邮箱格式基本合法? {isValidEmail}");
    }
}

// 运行判断示例
var quantifierDemo = new QuantifierExamples();
quantifierDemo.DemonstrateQuantifiers();
```

### 3.6 聚合函数（Count/Sum/Average/Max/Min）

#### 示例7：统计分析

```csharp
public class AggregationExamples
{
    public void DemonstrateAggregation()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("=== 聚合函数示例 ===\n");

        // Count：计数
        int totalProducts = products.Count();
        int onSaleCount = products.Count(p => p.IsOnSale);
        int phoneCount = products.Count(p => p.Category == "手机");

        Console.WriteLine("--- 商品统计 ---");
        Console.WriteLine($"总商品数: {totalProducts}");
        Console.WriteLine($"促销商品: {onSaleCount} ({(double)onSaleCount/totalProducts:P1})");
        Console.WriteLine($"手机数量: {phoneCount}");

        // Sum：求和
        decimal totalValue = products.Sum(p => p.Price * p.Stock);  // 库存总价值
        int totalStock = products.Sum(p => p.Stock);
        decimal totalOrderAmount = orders.Sum(o => o.Amount);

        Console.WriteLine($"\n--- 金额统计 ---");
        Console.WriteLine($"库存总价值: ¥{totalValue:N2}");
        Console.WriteLine($"总库存量: {totalStock} 件");
        Console.WriteLine($"订单总金额: ¥{totalOrderAmount:N2}");

        // Average：平均值
        double avgPrice = products.Average(p => (double)p.Price);
        double avgRating = products.Average(p => p.Rating);
        double avgOrderValue = orders.Average(o => (double)o.Amount);

        Console.WriteLine($"\n--- 平均值 ---");
        Console.WriteLine($"平均单价: ¥{avgPrice:N2}");
        Console.WriteLine($"平均评分: {avgRating:F2}/5.0");
        Console.WriteLine($"平均订单金额: ¥{avgOrderValue:N2}");

        // Max/Min：最大最小值
        decimal maxPrice = products.Max(p => p.Price);
        decimal minPrice = products.Min(p => p.Price);
        var mostExpensive = products.First(p => p.Price == maxPrice);
        var cheapest = products.First(p => p.Price == minPrice);

        Console.WriteLine($"\n--- 极值 ---");
        Console.WriteLine($"最高价: ¥{maxPrice:N2} ({mostExpensive.Name})");
        Console.WriteLine($"最低价: ¥{minPrice:N2} ({cheapest.Name})");
        Console.WriteLine($"最大订单: ¥{orders.Max(o => o.Amount):N2}");
        Console.WriteLine($"最小订单: ¥{orders.Min(o => o.Amount):N2}");

        // 综合统计报告
        Console.WriteLine($"\n{'=',50}");
        Console.WriteLine("  📊 商品数据分析报告");
        Console.WriteLine($"{'=',50}");
        Console.WriteLine($"商品种类: {products.Select(p => p.Category).Distinct().Count()} 个分类");
        Console.WriteLine($"价格区间: ¥{minPrice:N2} - ¥{maxPrice:N2}");
        Console.WriteLine($"价格中位数: ¥{GetMedian(products.Select(p => p.Price).ToList()):N2}");
        Console.WriteLine($"库存周转率估算: {CalculateTurnoverRate(products):F1} 次/年");
        Console.WriteLine($"高评分商品占比: {(double)products.Count(p => p.Rating >= 4.7)/products.Count():P1}");
    }

    private decimal GetMedian(List<decimal> values)
    {
        values.Sort();
        int count = values.Count;
        if (count % 2 == 0)
            return (values[count / 2 - 1] + values[count / 2]) / 2;
        return values[count / 2];
    }

    private double CalculateTurnoverRate(List<Product> products)
    {
        // 简化计算：假设年销售额/(平均库存*平均成本)
        decimal avgStock = products.Average(p => p.Stock);
        decimal avgCost = products.Average(p => p.Price * 0.6m);  // 假设成本是售价60%
        decimal annualSales = products.Sum(p => p.Price * 50);  // 假设每件年销量50

        return avgCost > 0 ? (double)(annualSales / (avgStock * avgCost)) : 0;
    }
}

// 运行聚合示例
var aggDemo = new AggregationExamples();
aggDemo.DemonstrateAggregation();
```

### 3.7 GroupBy - 分组

`.GroupBy()` 将集合按照指定键分组，功能强大！

#### 示例8：分组统计

```csharp
public class GroupByExamples
{
    public void DemonstrateGroupBy()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("=== GroupBy 分组示例 ===\n");

        // 场景1：按商品分类分组
        var byCategory = products.GroupBy(p => p.Category);

        Console.WriteLine("【按分类分组】");
        foreach (var group in byCategory)
        {
            Console.WriteLine($"\n▶ 分类: {group.Key} ({group.Count()}个商品)");
            Console.WriteLine($"  平均价格: ¥{group.Average(p => p.Price):N2}");
            Console.WriteLine($"  总库存: {group.Sum(p => p.Stock)} 件");
            Console.WriteLine($"  最高评分: {group.Max(p => p.Rating):F1}");

            foreach (var item in group)
                Console.WriteLine($"    - {item.Name}: ¥{item.Price:N2}");
        }

        // 场景2：按订单状态分组
        var byStatus = orders.GroupBy(o => o.Status);
        Console.WriteLine("\n\n【按订单状态分组】");
        foreach (var group in byStatus)
        {
            decimal total = group.Sum(o => o.Amount);
            Console.WriteLine($"{group.Key,-6}: {group.Count(),3}笔 | 总额: ¥{total,10:N2} | 均: ¥{group.Average(o => o.Amount):N2}");
        }

        // 场景3：按城市分组（客户地域分布）
        var byCity = orders.GroupBy(o => o.City).OrderByDescending(g => g.Sum(o => o.Amount));
        Console.WriteLine("\n\n【按城市分组（消费排行）】");
        foreach (var group byCity)
        {
            Console.WriteLine($"{group.Key,-6}: {group.Count(),2}位客户 | 消费总额: ¥{group.Sum(o => o.Amount),10:N2}");
        }

        // 场景4：按价格区间分组
        var byPriceRange = products.GroupBy(p =>
        {
            if (p.Price < 2000) return "经济型 (<2000)";
            if (p.Price < 7000) return "主流型 (2000-7000)";
            if (p.Price < 12000) return "高端型 (7000-12000)";
            return "旗舰型 (>12000)";
        });

        Console.WriteLine("\n\n【按价格区间分组】");
        foreach (var group in byPriceRange)
        {
            Console.WriteLine($"{group.Key,-22}: {group.Count(),2}个商品");
        }

        // 场景5：复合键分组（按分类和是否促销同时分组）
        var complexGroup = products.GroupBy(p => new { p.Category, p.IsOnSale });
        Console.WriteLine("\n\n【复合分组: 分类+促销状态】");
        foreach (var group in complexGroup)
        {
            string saleStatus = group.Key.IsOnSale ? "促销中" : "正常";
            Console.WriteLine($"[{group.Key.Category,-6}-{saleStatus,-4}] 数量:{group.Count(),2} 均价:¥{group.Average(p=>p.Price),8:N0}");
        }
    }
}

// 运行分组示例
var groupDemo = new GroupByExamples();
groupDemo.DemonstrateGroupBy();
```

### 3.8 Take / Skip / TakeWhile / SkipWhile - 分页与截取

#### 示例9：分页实现

```csharp
public class PaginationExamples
{
    public void DemonstratePagination()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("=== Take/Skip 分页示例 ===\n");

        // 场景1：简单的Take（取前N个）
        Console.WriteLine("【最贵的5个商品】");
        var top5Expensive = products.OrderByDescending(p => p.Price).Take(5);
        foreach (var p in top5Expensive)
            Console.WriteLine($"  {p.Name,-22} ¥{p.Price,8:N2}");

        // 场景2：Skip+Take实现分页
        int pageSize = 3;
        int pageNumber = 2;  // 第2页（从0开始算第1页）

        var pageData = products
            .OrderBy(p => p.Name)
            .Skip(pageNumber * pageSize)  // 跳过前pageSize*(pageNumber)个
            .Take(pageSize);              // 取pageSize个

        Console.WriteLine($"\n【第{pageNumber + 1}页 (每页{pageSize}个)】");
        foreach (var p in pageData)
            Console.WriteLine($"  {p.Name}");

        // 场景3：完整的分页器实现
        Console.WriteLine("\n【完整分页展示】");
        DisplayPaginatedResults(orders.OrderBy(o => o.OrderDate), 1, 3);  // 第1页，每页3条
        DisplayPaginatedResults(orders.OrderBy(o => o.OrderDate), 2, 3);  // 第2页
        DisplayPaginatedResults(orders.OrderBy(o => o.OrderDate), 3, 3);  // 第3页

        // 场景4：TakeWhile/SkipWhile（条件截取）
        var sortedByPrice = products.OrderBy(p => p.Price);
        var affordable = sortedByPrice.TakeWhile(p => p.Price < 5000);  // 从头开始，直到不满足条件
        Console.WriteLine("\n【价格<5000的商品（从头开始连续）】");
        foreach (var p in affordable)
            Console.WriteLine($"  {p.Name}: ¥{p.Price:N2}");

        // 场景5：实际Web开发中的分页场景
        Console.WriteLine("\n【模拟API分页请求】");
        SimulateApiPagination(orders, page: 1, size: 4);
        SimulateApiPagination(orders, page: 2, size: 4);
    }

    private void DisplayPaginatedResults<T>(IEnumerable<T> source, int page, int pageSize)
    {
        var items = source.Skip((page - 1) * pageSize).Take(pageSize).ToList();
        int totalCount = source.Count();
        int totalPages = (int)Math.Ceiling((double)totalCount / pageSize);

        Console.WriteLine($"\n--- 第 {page}/{totalPages} 页 ---");
        foreach (var item in items)
            Console.WriteLine($"  {item}");

        if (!items.Any())
            Console.WriteLine("  （本页无数据）");

        Console.WriteLine($"显示: {items.Count}/{totalCount} 条记录");
    }

    private void SimulateApiPagination(IEnumerable<Order> orders, int page, int size)
    {
        var query = orders.AsQueryable();  // 在真实项目中这通常是数据库查询

        var pagedResult = new
        {
            Page = page,
            PageSize = size,
            TotalCount = query.Count(),
            Items = query
                .OrderByDescending(o => o.OrderDate)
                .Skip((page - 1) * size)
                .Take(size)
                .ToList()
        };

        int totalPages = (int)Math.Ceiling((double)pagedResult.TotalCount / size);

        Console.WriteLine($"\nGET /api/orders?page={page}&size={size}");
        Console.WriteLine($"Response:");
        Console.WriteLine($"{{");
        Console.WriteLine($"  \"page\": {pagedResult.Page},");
        Console.WriteLine($"  \"pageSize\": {pagedResult.PageSize},");
        Console.WriteLine($"  \"totalCount\": {pagedResult.TotalCount},");
        Console.WriteLine($"  \"totalPages\": {totalPages},");

        Console.WriteLine($"  \"items\": [");
        foreach (var order in pagedResult.Items)
        {
            Console.WriteLine($"    {{\"id\": {order.OrderId}, \"customer\": \"{order.CustomerName}\", " +
                           $"\"amount\": {order.Amount}}},");
        }
        Console.WriteLine($"  ]");
        Console.WriteLine($"}}");
    }
}

// 运行分页示例
var paginationDemo = new PaginationExamples();
paginationDemo.DemonstratePagination();
```

---

## 四、综合实战案例

### 案例1：电商数据仪表盘

```csharp
public class EcommerceDashboard
{
    public void GenerateDashboard()
    {
        var products = TestData.GetProducts();
        var orders = TestData.GetOrders();

        Console.WriteLine("╔══════════════════════════════════════════╗");
        Console.WriteLine("║      📊 电商运营数据仪表盘 v1.0         ║");
        Console.WriteLine("╚══════════════════════════════════════════╝\n");

        // ===== 核心指标卡片 =====
        PrintKPIs(orders, products);

        // ===== 商品分析 =====
        AnalyzeProducts(products);

        // ===== 订单分析 =====
        AnalyzeOrders(orders);

        // ===== 地域分布 =====
        AnalyzeGeography(orders);

        // ===== 趋势洞察 =====
        GenerateInsights(products, orders);
    }

    private void PrintKPIs(List<Order> orders, List<Product> products)
    {
        Console.WriteLine("┌──────────────────────────────────────────┐");
        Console.WriteLine("│           🎯 核心业务指标                 │");
        Console.WriteLine("└──────────────────────────────────────────┘\n");

        decimal totalRevenue = orders
            .Where(o => o.Status == "已完成")
            .Sum(o => o.Amount);

        decimal pendingRevenue = orders
            .Where(o => o.Status == "待付款")
            .Sum(o => o.Amount);

        double completionRate = orders.Count() > 0 ?
            (double)orders.Count(o => o.Status == "已完成") / orders.Count() * 100 : 0;

        double avgOrderValue = orders.Any() ?
            orders.Average(o => (double)o.Amount) : 0;

        Console.WriteLine($"┌─────────────┬────────────┬────────────┐");
        Console.WriteLine($"│ 总营收(完成) │ 待确认收入  │ 订单完成率  │");
        Console.WriteLine($"├─────────────┼────────────┼────────────┤");
        Console.WriteLine($"│ ¥{totalRevenue,9:N0} │ ¥{pendingRevenue,8:N0} │ {completionRate,7:F1}%   │");
        Console.WriteLine($"├─────────────┼────────────┼────────────┤");
        Console.WriteLine($"│ 平均客单价   │ 总订单数    │ 商品总数    │");
        Console.WriteLine($"├─────────────┼────────────┼────────────┤");
        Console.WriteLine($"│ ¥{avgOrderValue,9:N2} │ {orders.Count(),9} │ {products.Count(),9} │");
        Console.WriteLine($"└─────────────┴────────────┴────────────┘\n");
    }

    private void AnalyzeProducts(List<Product> products)
    {
        Console.WriteLine("┌──────────────────────────────────────────┐");
        Console.WriteLine("│           📦 商品分析                     │");
        Console.WriteLine("└──────────────────────────────────────────┘\n");

        // TOP5热销（按库存周转假设）
        var topProducts = products
            .OrderByDescending(p => p.Rating)
            .ThenBy(p => p.Stock)
            .Take(5);

        Console.WriteLine("【TOP5 高评分商品】");
        Console.WriteLine($"{"排名",-4} {"商品名",-18} {"评分",5} {"价格",10} {"库存",5}");
        Console.WriteLine(new string('-', 48));

        int rank = 1;
        foreach (var p in topProducts)
        {
            Console.WriteLine($"{rank++,-4} {p.Name,-18} {"★"+p.Rating,-5} ¥{p.Price,8:N2} {p.Stock,5}");
        }

        // 库存预警
        var lowStockProducts = products
            .Where(p => p.Stock < 50)
            .OrderBy(p => p.Stock);

        if (lowStockProducts.Any())
        {
            Console.WriteLine($"\n⚠️  库存预警 ({lowStockProducts.Count()}个商品):");
            foreach (var p in lowStockProducts)
            {
                string urgency = p.Stock < 10 ? "🔴 紧急" : "🟡 注意";
                Console.WriteLine($"  {urgency} {p.Name,-18} 剩余: {p.Stock,3}件");
            }
        }
    }

    private void AnalyzeOrders(List<Order> orders)
    {
        Console.WriteLine("\n┌──────────────────────────────────────────┐");
        Console.WriteLine("│           🛒 订单分析                     │");
        Console.WriteLine("└──────────────────────────────────────────┘\n");

        // 订单状态分布
        var statusGroups = orders.GroupBy(o => o.Status);
        Console.WriteLine("【订单状态分布】");
        foreach (var group in statusGroups.OrderBy(g => g.Key))
        {
            int count = group.Count();
            decimal amount = group.Sum(o => o.Amount);
            double percentage = (double)count / orders.Count() * 100;

            string bar = new string('█', (int)(percentage / 5));
            Console.WriteLine($"  {group.Key,-6} {count,3}笔 ({percentage,5:F1}%) {bar} ¥{amount:N0}");
        }

        // 大客户识别
        var topCustomers = orders
            .GroupBy(o => o.CustomerName)
            .Select(g => new
            {
                Customer = g.Key,
                Orders = g.Count(),
                TotalSpent = g.Sum(o => o.Amount),
                AvgOrder = g.Average(o => o.Amount)
            })
            .OrderByDescending(c => c.TotalSpent)
            .Take(5);

        Console.WriteLine("\n【TOP5 高价值客户】");
        Console.WriteLine($"{"客户",-6} {"订单数",5} {"总消费",10} {"客单价",10}");
        Console.WriteLine(new string('-', 36));

        foreach (var c in topCustomers)
        {
            Console.WriteLine($"{c.Customer,-6} {c.Orders,5} ¥{c.TotalSpent,9:N0} ¥{c.AvgOrder,9:N0}");
        }
    }

    private void AnalyzeGeography(List<Order> orders)
    {
        Console.WriteLine("\n┌──────────────────────────────────────────┐");
        Console.WriteLine("│           🗺️  地域分布                     │");
        Console.WriteLine("└──────────────────────────────────────────┘\n");

        var cityStats = orders
            .GroupBy(o => o.City)
            .Select(g => new
            {
                City = g.Key,
                OrderCount = g.Count(),
                Revenue = g.Sum(o => o.Amount),
                Customers = g.Select(o => o.CustomerName).Distinct().Count()
            })
            .OrderByDescending(c => c.Revenue);

        Console.WriteLine($"{"城市",-6} {"客户数",5} {"订单数",5} {"营收",12} {"占比",6}");
        Console.WriteLine(new string('-', 40));

        decimal totalRevenue = orders.Sum(o => o.Amount);
        foreach (var city in cityStats)
        {
            double pct = (double)city.Revenue / totalRevenue * 100;
            Console.WriteLine($"{city.City,-6} {city.Customers,5} {city.OrderCount,5} ¥{city.Revenue,10:N0} {pct,5:F1}%");
        }
    }

    private void GenerateInsights(List<Product> products, List<Order> orders)
    {
        Console.WriteLine("\n┌──────────────────────────────────────────┐");
        Console.WriteLine("│           💡 智能洞察                     │");
        Console.WriteLine("└──────────────────────────────────────────┘\n");

        var insights = new List<string>();

        // 洞察1：促销效果
        var onSaleProducts = products.Where(p => p.IsOnSale);
        var onSaleRevenue = orders
            .Where(o => onSaleProducts.Any(p => p.Price <= o.Amount))
            .Sum(o => o.Amount);

        if (onSaleProducts.Any())
        {
            insights.Add($"✓ 当前有 {onSaleProducts.Count()} 个商品正在促销");
            insights.Add($"  促销商品均价: ¥{onSaleProducts.Average(p => p.Price):N0}");
        }

        // 洞察2：库存风险
        var criticalStock = products.Count(p => p.Stock < 20);
        if (criticalStock > 0)
        {
            insights.Add($"\n⚠️  {criticalStock} 个商品库存严重不足(<20件)");
            insights.Add("  建议：立即联系供应商补货");
        }

        // 洞察3：客户留存
        var repeatBuyers = orders
            .GroupBy(o => o.CustomerName)
            .Count(g => g.Count() > 1);

        double retentionRate = (double)repeatBuyers / orders.Select(o => o.CustomerName).Distinct().Count() * 100;

        insights.Add($"\n📈 复购率: {retentionRate:F1}%");
        if (retentionRate > 30)
            insights.Add("  客户忠诚度较高，继续保持！");
        else
            insights.Add("  建议推出会员积分计划提升复购");

        // 洞察4：价格带分析
        var midRange = products.Count(p => p.Price >= 2000 && p.Price <= 8000);
        insights.Add($"\n💰 主力价格带(¥2000-8000)商品: {midRange}个，占{(double)midRange/products.Count():P0}");

        foreach (var insight in insights)
        {
            Console.WriteLine(insight);
        }
    }
}

// 运行仪表盘
var dashboard = new EcommerceDashboard();
dashboard.GenerateDashboard();
```

---

## 五、查询语法 vs 方法语法

LINQ提供两种写法：

### 5.1 对比示例

```csharp
public class QuerySyntaxDemo
{
    public void CompareSyntaxes()
    {
        var products = TestData.GetProducts();

        // ========== 方法语法（推荐，更灵活）==========
        var methodSyntax = products
            .Where(p => p.Price > 3000)
            .OrderBy(p => p.Name)
            .Select(p => new { p.Name, p.Price });

        // ========== 查询语法（类似SQL，更直观）==========
        var querySyntax =
            from p in products
            where p.Price > 3000
            orderby p.Name
            select new { p.Name, p.Price };

        // 两者结果完全相同！
        Console.WriteLine("方法语法结果:");
        foreach (var item in methodSyntax)
            Console.WriteLine($"  {item.Name}: ¥{item.Price:N2}");

        Console.WriteLine("\n查询语法结果:");
        foreach (var item in querySyntax)
            Console.WriteLine($"  {item.Name}: ¥{item.Price:N2}");
    }

    // 复杂查询对比
    public void ComplexQueryComparison()
    {
        var orders = TestData.GetOrders();

        // 方法语法：查找消费总额>15000的客户及其订单详情
        var methodResult = orders
            .GroupBy(o => o.CustomerName)
            .Where(g => g.Sum(o => o.Amount) > 15000)
            .SelectMany(g => g.Select(o => new
            {
                Customer = g.Key,
                o.OrderId,
                o.Amount,
                o.Status,
                TotalSpent = g.Sum(x => x.Amount)
            }))
            .OrderByDescending(x => x.TotalSpent)
            .ThenBy(x => x.OrderDate);

        // 查询语法：同样的逻辑
        var queryResult =
            from o in orders
            group o by o.CustomerName into g
            where g.Sum(o => o.Amount) > 15000
            from o in g
            orderby g.Sum(o => o.Amount) descending, o.OrderDate
            select new
            {
                Customer = g.Key,
                o.OrderId,
                o.Amount,
                o.Status,
                TotalSpent = g.Sum(x => x.Amount)
            };

        Console.WriteLine("【高价值客户订单明细】\n");
        foreach (var item in queryResult)
        {
            Console.WriteLine($"{item.Customer,-6} #{item.OrderId} ¥{item.Amount,8:N0} " +
                           $"[{item.Status}] (总计: ¥{item.TotalSpent:N0})");
        }
    }
}

// 运行对比
var syntaxDemo = new QuerySyntaxDemo();
syntaxDemo.CompareSyntaxes();
syntaxDemo.ComplexQueryComparison();
```

### 5.2 选择建议

| 场景 | 推荐语法 | 原因 |
|------|----------|------|
| 简单查询 | 查询语法 | 更接近SQL，易读 |
| 复杂链式调用 | **方法语法** | 更灵活，支持更多操作 |
| 需要Lambda做复杂逻辑 | **方法语法** | Lambda更强大 |
| 团队成员熟悉SQL | 查询语法 | 降低学习成本 |
| 需要调试中间结果 | **方法语法** | 可以逐步查看 |

---

## 六、性能注意事项

### 6.1 延迟执行 vs 立即执行

```csharp
public class ExecutionDemo
{
    public void DemonstrateExecution()
    {
        var products = TestData.GetProducts();

        // 延迟执行（默认）：定义时不执行，遍历时才执行
        var query = products.Where(p => p.Price > 5000);
        Console.WriteLine("查询已定义（尚未执行）");

        // 此时才真正执行
        Console.WriteLine($"\n开始遍历，触发执行...");
        foreach (var p in query)
        {
            Console.WriteLine($"  找到: {p.Name}");
        }

        // 立即执行：调用ToList()/ToArray()/Count()等立即执行
        var immediateResult = products
            .Where(p => p.Price > 5000)
            .ToList();  // ← 立即执行并保存到内存

        Console.WriteLine($"\n立即执行完成，结果已缓存: {immediateResult.Count} 条");

        // 性能陷阱示例
        Console.WriteLine("\n--- 性能陷阱演示 ---");
        PerformanceTrapExample(products);
    }

    private void PerformanceTrapExample(List<Product> products)
    {
        // ❌ 陷阱：多次枚举导致重复计算
        var expensiveProducts = products.Where(p =>
        {
            Console.WriteLine($"  正在检查: {p.Name}");  // 会打印多次！
            return p.Price > 5000;
        });

        Console.WriteLine("第一次Count:");
        int count1 = expensiveProducts.Count();  // 第一次执行

        Console.WriteLine("\n第二次Count:");
        int count2 = expensiveProducts.Count();  // 又执行了一次！

        Console.WriteLine($"\n结果相同但执行了两次查询!");

        // ✅ 改进：使用ToList()缓存结果
        var cachedProducts = products.Where(p => p.Price > 5000).ToList();

        Console.WriteLine("\n使用ToList()缓存后:");
        Console.WriteLine($"Count: {cachedProducts.Count}");  // 从内存读取
        Console.WriteLine($"Count: {cachedProducts.Count}");  // 还是从内存读取，不再重新查询
    }
}

// 运行执行演示
var execDemo = new ExecutionDemo();
execDemo.DemonstrateExecution();
```

### 6.2 最佳实践清单

- [ ] **大数据集考虑分页**：避免一次性加载全部数据
- [ ] **合理使用ToList()**：多次使用时先缓存
- [ ] **注意闭包变量捕获**：Lambda中的外部变量会在执行时取值
- [ ] **优先使用方法语法**：更灵活且易于组合
- [ ] **复杂查询拆分**：提高可读性和可维护性
- [ ] **考虑使用AsQueryable()**：数据库场景下让LINQ to SQL优化

---

## 七、练习题

### 练习1：学生成绩管理系统
**题目**：给定学生成绩列表（姓名、科目、分数），使用LINQ完成：
1. 找出数学成绩>80的学生
2. 计算每个学生的平均分
3. 找出各科最高分
4. 统计各分数段人数（90-100优秀，80-89良好...）

<details>
<summary>查看答案</summary>

```csharp
public class Score
{
    public string StudentName { get; set; }
    public string Subject { get; set; }
    public int ScoreValue { get; set; }
}

public class StudentGradeManager
{
    public void AnalyzeGrades(List<Score> scores)
    {
        // 1. 数学成绩>80的学生
        var mathHighScorers = scores
            .Where(s => s.Subject == "数学" && s.ScoreValue > 80)
            .OrderByDescending(s => s.ScoreValue);

        Console.WriteLine("【数学成绩>80的学生】");
        foreach (var s in mathHighScorers)
            Console.WriteLine($"  {s.StudentName}: {s.ScoreValue}分");

        // 2. 每个学生的平均分
        var studentAverages = scores
            .GroupBy(s => s.StudentName)
            .Select(g => new
            {
                Student = g.Key,
                Average = g.Average(s => s.ScoreValue),
                Subjects = g.Count()
            })
            .OrderByDescending(g => g.Average);

        Console.WriteLine("\n【学生平均分排名】");
        foreach (var s in studentAverages)
            Console.WriteLine($"  {s.Student,-8} 均分: {s.Average,6:F1} ({s.Subjects}科)");

        // 3. 各科最高分
        var subjectTops = scores
            .GroupBy(s => s.Subject)
            .Select(g => new
            {
                Subject = g.Key,
                TopScore = g.Max(s => s.ScoreValue),
                TopStudent = g.First(s => s.ScoreValue == g.Max(x => x.ScoreValue)).StudentName
            });

        Console.WriteLine("\n【各科最高分】");
        foreach (var s in subjectTops)
            Console.WriteLine($"  {s.Subject,-6}: {s.TopStudent,-8} {s.TopScore}分");

        // 4. 分数段统计
        var gradeDistribution = scores
            .GroupBy(s =>
            {
                if (s.ScoreValue >= 90) return "优秀 (90-100)";
                if (s.ScoreValue >= 80) return "良好 (80-89)";
                if (s.ScoreValue >= 70) return "中等 (70-79)";
                if (s.ScoreValue >= 60) return "及格 (60-69)";
                return "不及格 (<60)";
            })
            .Select(g => new { Grade = g.Key, Count = g.Count() })
            .OrderByDescending(g => g.Grade.Contains("优秀") ? 1 :
                               g.Grade.Contains("良好") ? 2 :
                               g.Grade.Contains("中等") ? 3 :
                               g.Grade.Contains("及格") ? 4 : 5);

        Console.WriteLine("\n【分数段分布】");
        foreach (var g in gradeDistribution)
        {
            string bar = new string('█', g.Count);
            Console.WriteLine($"  {g.Grade,-16} {g.Count,3}人 {bar}");
        }
    }
}

// 测试数据
var testScores = new List<Score>
{
    new Score { StudentName = "张三", Subject = "语文", ScoreValue = 92 },
    new Score { StudentName = "张三", Subject = "数学", ScoreValue = 85 },
    new Score { StudentName = "张三", Subject = "英语", ScoreValue = 78 },
    new Score { StudentName = "李四", Subject = "语文", ScoreValue = 88 },
    new Score { StudentName = "李四", Subject = "数学", ScoreValue = 95 },
    new Score { StudentName = "李四", Subject = "英语", ScoreValue = 91 },
    new Score { StudentName = "王五", Subject = "语文", ScoreValue = 76 },
    new Score { StudentName = "王五", Subject = "数学", ScoreValue = 62 },
    new Score { StudentName = "王五", Subject = "英语", ScoreValue = 55 },
    new Score { StudentName = "赵六", Subject = "语文", ScoreValue = 98 },
    new Score { StudentName = "赵六", Subject = "数学", ScoreValue = 82 },
    new Score { StudentName = "赵六", Subject = "英语", ScoreValue = 88 }
};

var manager = new StudentGradeManager();
manager.AnalyzeGrades(testScores);
```
</details>

### 练习2：文本分析工具
**题目**：给定一段英文文本，使用LINQ统计：
1. 单词总数
2. 不重复单词数
3. 出现频率最高的Top10单词（忽略大小写）
4. 平均单词长度
5. 最长的单词

<details>
<summary>查看答案</summary>

```csharp
public class TextAnalyzer
{
    public void Analyze(string text)
    {
        // 预处理：分割单词并清理
        var words = text
            .ToLower()
            .Split(new[] { ' ', '\n', '\r', '\t', '.', ',', '!', '?', ';', ':',
                          '"', '(', ')', '[', ']', '{', '}', '-', '_' },
                   StringSplitOptions.RemoveEmptyEntries)
            .ToList();

        Console.WriteLine("=== 文本分析报告 ===\n");

        // 1. 单词总数
        Console.WriteLine($"总单词数: {words.Count}");

        // 2. 不重复单词数
        var uniqueWords = words.Distinct().Count();
        Console.WriteLine($"不重复单词数: {uniqueWords}");

        // 3. Top10高频词
        var topWords = words
            .GroupBy(w => w)
            .OrderByDescending(g => g.Count())
            .ThenBy(g => g.Key)
            .Take(10)
            .Select(g => new { Word = g.Key, Count = g.Count() });

        Console.WriteLine("\n【Top 10 高频词】");
        foreach (var w in topWords)
        {
            string bar = new string('█', w.Count);
            Console.WriteLine($"  {w.Word,-15} {w.Count,4} {bar}");
        }

        // 4. 平均单词长度
        double avgLength = words.Average(w => w.Length);
        Console.WriteLine($"\n平均单词长度: {avgLength:F2} 字符");

        // 5. 最长的单词（可能有多个）
        int maxLength = words.Max(w => w.Length);
        var longestWords = words
            .Where(w => w.Length == maxLength)
            .Distinct()
            .OrderBy(w => w);

        Console.WriteLine($"\n最长单词({maxLength}字符):");
        Console.WriteLine($"  [{string.Join(", ", longestWords)}]");
    }
}

// 测试
string sampleText = @"
LINQ is a powerful feature in C# that allows developers to write queries
in a declarative way. With LINQ, you can filter, sort, project, and aggregate
data from various sources like arrays, lists, databases, and XML documents.
LINQ makes code more readable and maintainable. Learning LINQ is essential
for modern C# development. LINQ supports both query syntax and method syntax.
";

var analyzer = new TextAnalyzer();
analyzer.Analyze(sampleText);
```
</details>

### 练习3：员工数据报表系统
**题目**：给定员工列表（姓名、部门、职位、薪资、入职年份），生成月度报表：
1. 各部门人数和平均薪资
2. 薪资最高的Top5员工
3. 各职位的薪资范围（最高/最低/平均）
4. 入职年限分布（0-1年，1-3年，3-5年，5年以上）
5. 找出薪资异常高或低的员工（高于平均值50%或低于50%）

<details>
<summary>查看答案</summary>

```csharp
public class Employee
{
    public string Name { get; set; }
    public string Department { get; set; }
    public string Position { get; set; }
    public decimal Salary { get; set; }
    public int HireYear { get; set; }
}

public class EmployeeReportGenerator
{
    public void GenerateReport(List<Employee> employees)
    {
        Console.WriteLine("╔═══════════════════════════════════════╗");
        Console.WriteLine("║      员工数据月度报表                  ║");
        Console.WriteLine("╚═══════════════════════════════════════╝\n");

        // 1. 部门统计
        var deptStats = employees
            .GroupBy(e => e.Department)
            .Select(d => new
            {
                Department = d.Key,
                Headcount = d.Count(),
                AvgSalary = d.Average(e => e.Salary),
                TotalPayroll = d.Sum(e => e.Salary)
            })
            .OrderByDescending(d => d.Headcount);

        Console.WriteLine("【部门统计】");
        Console.WriteLine($"{"部门",-10} {"人数",5} {"平均薪资",12} {"薪资总额",14}");
        Console.WriteLine(new string('-', 45));
        foreach (var d in deptStats)
        {
            Console.WriteLine($"{d.Department,-10} {d.Headcount,5} ¥{d.AvgSalary,10:N0} ¥{d.TotalPayroll,12:N0}");
        }

        // 2. Top5高薪员工
        var topEarners = employees
            .OrderByDescending(e => e.Salary)
            .Take(5);

        Console.WriteLine("\n【Top5 高薪员工】");
        Console.WriteLine($"{"排名",-4} {"姓名",-8} {"部门",-8} {"职位",-10} {"薪资",12}");
        Console.WriteLine(new string("-", 46));
        int rank = 1;
        foreach (var emp in topEarners)
        {
            Console.WriteLine($"{rank++,-4} {emp.Name,-8} {emp.Department,-8} {emp.Position,-10} ¥{emp.Salary,11:N0}");
        }

        // 3. 职位薪资分析
        var positionAnalysis = employees
            .GroupBy(e => e.Position)
            .Select(p => new
            {
                Position = p.Key,
                MinSalary = p.Min(e => e.Salary),
                MaxSalary = p.Max(e => e.Salary),
                AvgSalary = p.Average(e => e.Salary),
                Count = p.Count()
            })
            .OrderByDescending(p => p.AvgSalary);

        Console.WriteLine("\n【职位薪资分析】");
        Console.WriteLine($"{"职位",-12} {"人数",4} {"最低",10} {"平均",10} {"最高",10}");
        Console.WriteLine(new string("-", 50));
        foreach (var p in positionAnalysis)
        {
            Console.WriteLine($"{p.Position,-12} {p.Count,4} ¥{p.MinSalary,9:N0} ¥{p.AvgSalary,9:N0} ¥{p.MaxSalary,9:N0}");
        }

        // 4. 入职年限分布
        int currentYear = DateTime.Now.Year;
        var tenureDist = employees
            .GroupBy(e =>
            {
                int years = currentYear - e.HireYear;
                if (years <= 1) return "0-1年";
                if (years <= 3) return "1-3年";
                if (years <= 5) return "3-5年";
                return "5年以上";
            })
            .Select(g => new { Tenure = g.Key, Count = g.Count() });

        Console.WriteLine("\n【入职年限分布】");
        foreach (var t in tenureDist)
        {
            double pct = (double)t.Count / employees.Count() * 100;
            string bar = new string('█', (int)(pct / 2));
            Console.WriteLine($"  {t.Tenure,-8} {t.Count,3}人 ({pct,5:F1}%) {bar}");
        }

        // 5. 薪资异常检测
        decimal overallAvg = employees.Average(e => e.Salary);
        decimal upperThreshold = overallAvg * 1.5m;
        decimal lowerThreshold = overallAvg * 0.5m;

        var outliers = employees
            .Where(e => e.Salary > upperThreshold || e.Salary < lowerThreshold)
            .Select(e => new
            {
                e.Name,
                e.Salary,
                Deviation = (double)((e.Salary - overallAvg) / overallAvg * 100),
                Type = e.Salary > upperThreshold ? "偏高" : "偏低"
            })
            .OrderByDescending(o => Math.Abs(o.Deviation));

        Console.WriteLine($"\n【薪资异常员工】(平均: ¥{overallAvg:N0})");
        Console.WriteLine($"正常范围: ¥{lowerThreshold:N0} - ¥{upperThreshold:N0}\n");

        if (outliers.Any())
        {
            foreach (var o in outliers)
            {
                string icon = o.Type == "偏高" ? "↑" : "↓";
                Console.WriteLine($"  {icon} {o.Name,-8} ¥{o.Salary,10:N0} ({o.Deviation:+F1;-F1}%)");
            }
        }
        else
        {
            Console.WriteLine("  ✓ 所有员工薪资都在正常范围内");
        }
    }
}

// 测试
var employees = new List<Employee>
{
    new Employee { Name = "张三", Department = "技术部", Position = "高级工程师", Salary = 35000, HireYear = 2019 },
    new Employee { Name = "李四", Department = "技术部", Position = "工程师", Salary = 22000, HireYear = 2021 },
    new Employee { Name = "王五", Department = "市场部", Position = "经理", Salary = 28000, HireYear = 2018 },
    new Employee { Name = "赵六", Department = "市场部", Position = "专员", Salary = 12000, HireYear = 2023 },
    new Employee { Name = "钱七", Department = "财务部", Position = "主管", Salary = 30000, HireYear = 2017 },
    new Employee { Name = "孙八", Department = "财务部", Position = "会计", Salary = 15000, HireYear = 2022 },
    new Employee { Name = "周九", Department = "技术部", Position = "架构师", Salary = 55000, HireYear = 2015 },  // 异常偏高
    new Employee { Name = "吴十", Department = "人事部", Position = "助理", Salary = 6000, HireYear = 2024 }   // 异常偏低
};

var reportGen = new EmployeeReportGenerator();
reportGen.GenerateReport(employees);
```
</details>

### 练习4：进阶挑战 - 日志分析器
**题目**：解析Web服务器日志，使用LINQ完成：
1. 统计每小时请求数量
2. 找出访问最频繁的10个URL
3. 分析HTTP状态码分布
4. 识别IP地址访问频率（找出可能爬虫的IP）
5. 计算平均响应时间

<details>
<summary>查看答案</summary>

```csharp
public class LogEntry
{
    public DateTime Timestamp { get; set; }
    public string IpAddress { get; set; }
    public string Method { get; set; }
    public string Url { get; set; }
    public int StatusCode { get; set; }
    public long ResponseTimeMs { get; set; }
    public string UserAgent { get; set; }
}

public class LogAnalyzer
{
    public void AnalyzeLogs(List<LogEntry> logs)
    {
        Console.WriteLine("=== Web服务器日志分析报告 ===\n");

        DateTime startTime = logs.Min(l => l.Timestamp);
        DateTime endTime = logs.Max(l => l.Timestamp);
        TimeSpan duration = endTime - startTime;

        Console.WriteLine($"分析时间范围: {startTime:yyyy-MM-dd HH:mm} ~ {endTime:yyyy-MM-dd HH:mm}");
        Console.WriteLine($"日志总条数: {logs.Count}");
        Console.WriteLine($"时间跨度: {duration.TotalHours:F1} 小时\n");

        // 1. 每小时请求数
        var hourlyRequests = logs
            .GroupBy(l => l.Timestamp.Hour)
            .OrderBy(g => g.Key)
            .Select(g => new { Hour = g.Key, Count = g.Count() });

        Console.WriteLine("【每小时请求数量】");
        Console.WriteLine($"{"时间",8} {"请求数",8} {"可视化",20}");
        Console.WriteLine(new string('-', 38));
        foreach (var h in hourlyRequests)
        {
            string timeStr = $"{h.Hour:D2}:00";
            string bar = new string('█', Math.Min(h.Count / 5, 20));
            Console.WriteLine($"{timeStr,8} {h.Count,8} {bar,20}");
        }

        // 2. Top10 URL
        var topUrls = logs
            .GroupBy(l => l.Url)
            .OrderByDescending(g => g.Count())
            .Take(10)
            .Select(g => new { Url = g.Key, Count = g.Count(), AvgResponse = g.Average(l => l.ResponseTimeMs) });

        Console.WriteLine("\n【Top 10 热门URL】");
        Console.WriteLine($"{"#",3} {"访问次数",8} {"平均响应(ms)",12} {"URL",-35}");
        Console.WriteLine(new string("-", 62));
        int i = 1;
        foreach (var url in topUrls)
        {
            string displayUrl = url.Url.Length > 35 ? url.Url.Substring(0, 32) + "..." : url.Url;
            Console.WriteLine($"{i++,3} {url.Count,8} {url.AvgResponse,12:F0} {displayUrl,-35}");
        }

        // 3. HTTP状态码分布
        var statusCodes = logs
            .GroupBy(l => l.StatusCode)
            .OrderByDescending(g => g.Count())
            .Select(g => new { Code = g.Key, Count = g.Count(), Percentage = (double)g.Count() / logs.Count() * 100 });

        Console.WriteLine("\n【HTTP状态码分布】");
        foreach (var sc in statusCodes)
        {
            string desc = sc.Code switch
            {
                200 => "OK",
                301 => "重定向",
                304 => "未修改",
                400 => "错误请求",
                401 => "未授权",
                403 => "禁止访问",
                404 => "未找到",
                500 => "服务器错误",
                502 => "网关错误",
                503 => "服务不可用",
                _ => "其他"
            };

            string color = sc.Code < 400 ? "✓" : (sc.Code < 500 ? "⚠️" : "✗");
            Console.WriteLine($"  {color} {sc.Code} {desc,-10} {sc.Count,5}次 ({sc.Percentage:F1}%)");
        }

        // 4. IP访问频率（疑似爬虫检测）
        var ipStats = logs
            .GroupBy(l => l.IpAddress)
            .Select(g => new
            {
                IP = g.Key,
                RequestCount = g.Count(),
                UniqueUrls = g.Select(l => l.Url).Distinct().Count(),
                TimeSpan = (g.Max(l => l.Timestamp) - g.Min(l => l.Timestamp)).TotalMinutes,
                RequestsPerMinute = g.Count() / Math.Max((g.Max(l => l.Timestamp) - g.Min(l => l.Timestamp)).TotalMinutes, 1)
            })
            .OrderByDescending(ip => ip.RequestCount);

        Console.WriteLine("\n【Top 10 高频IP（爬虫嫌疑）】");
        var suspiciousIPs = ipStats.Take(10);
        Console.WriteLine($"{"IP地址",-18} {"请求次数",8} {"URL数",6} {"请求/分钟",10} {"风险评估",8}");
        Console.WriteLine(new string("-", 58));

        foreach (var ip in suspiciousIPs)
        {
            string risk = ip.RequestsPerMinute > 10 ? "🔴 高危" :
                         ip.RequestsPerMinute > 5 ? "🟠 中等" : "🟢 正常";

            Console.WriteLine($"{ip.IP,-18} {ip.RequestCount,8} {ip.UniqueUrls,6} {ip.RequestsPerMinute,10:F1} {risk,8}");
        }

        // 5. 响应时间统计
        double avgResponse = logs.Average(l => l.ResponseTimeMs);
        double maxResponse = logs.Max(l => l.ResponseTimeMs);
        double minResponse = logs.Min(l => l.ResponseTimeMs);
        double p95Response = logs.OrderBy(l => l.ResponseTimeMs)
                                 .Skip((int)(logs.Count * 0.95))
                                 .First().ResponseTimeMs;

        var responseDist = logs.GroupBy(l =>
        {
            if (l.ResponseTimeMs < 100) return "<100ms (快)";
            if (l.ResponseTimeMs < 500) return "100-500ms (良)";
            if (l.ResponseTimeMs < 1000) return "500ms-1s (一般)";
            return ">1s (慢)";
        }).Select(g => new { Range = g.Key, Count = g.Count(), Pct = (double)g.Count() / logs.Count() * 100 });

        Console.WriteLine("\n【响应时间统计】");
        Console.WriteLine($"  平均: {avgResponse:F0}ms");
        Console.WriteLine($"  最小: {minResponse:F0}ms");
        Console.WriteLine($"  最大: {maxResponse:F0}ms");
        Console.WriteLine($"  P95:  {p95Response:F0}ms (95%请求在此时间内完成)\n");

        Console.WriteLine("响应时间分布:");
        foreach (var r in responseDist)
        {
            string bar = new string('█', (int)(r.Pct / 2));
            Console.WriteLine($"  {r.Range,-17} {r.Count,5} ({r.Pct,5:F1}%) {bar}");
        }
    }
}

// 生成模拟日志数据
public static List<LogEntry> GenerateSampleLogs(int count = 1000)
{
    Random random = new Random();
    string[] urls = { "/", "/api/users", "/api/products", "/api/orders",
                     "/login", "/register", "/search", "/cart", "/checkout",
                     "/about", "/contact", "/faq" };
    string[] ips = { "192.168.1.100", "10.0.0.50", "172.16.0.25", "203.0.113.10",
                    "198.51.100.5", "192.168.1.101", "10.0.0.51" };  // 最后两个是"爬虫"
    int[] statusCodes = { 200, 200, 200, 200, 200, 200, 200, 304, 404, 500 };

    var logs = new List<LogEntry>();
    DateTime baseTime = DateTime.Now.AddHours(-2);

    for (int i = 0; i < count; i++)
    {
        // 爬虫IP产生更多请求
        string ip = random.NextDouble() < 0.15 ?
            ips[random.Next(ips.Length - 2)] :  // 正常IP
            ips[ips.Length - 1 - random.Next(2)]; // 爬虫IP

        logs.Add(new LogEntry
        {
            Timestamp = baseTime.AddMinutes(random.Next(120)),
            IpAddress = ip,
            Method = random.NextDouble() > 0.1 ? "GET" : "POST",
            Url = urls[random.Next(urls.Length)],
            StatusCode = statusCodes[random.Next(statusCodes.Length)],
            ResponseTimeMs = random.Next(10, 2000),
            UserAgent = "Mozilla/5.0"
        });
    }

    return logs.OrderBy(l => l.Timestamp).ToList();
}

// 运行日志分析
var sampleLogs = GenerateSampleLogs(1000);
var logAnalyzer = new LogAnalyzer();
logAnalyzer.AnalyzeLogs(sampleLogs);
```
</details>

### 练习5：终极挑战 - 构建简易ORM查询构建器
**题目**：设计一个简单的查询构建器类，支持链式调用，最终输出类似SQL的查询语句。支持Where/OrderBy/Take/Skip/Select操作。

<details>
<summary>查看答案</summary>

```csharp
/// <summary>
/// 简易查询构建器（教学用途）
/// </summary>
public class QueryBuilder<T>
{
    private IEnumerable<T> _source;
    private List<string> _conditions;
    private List<string> _orderByClauses;
    private int? _takeCount;
    private int? _skipCount;
    private string _selectClause;


    public QueryBuilder(IEnumerable<T> source)
    {
        _source = source;
        _conditions = new List<string>();
        _orderByClauses = new List<string>();
    }

    public QueryBuilder<T> Where(Func<T, bool> predicate, string description)
    {
        _conditions.Add(description);
        _source = _source.Where(predicate);
        return this;
    }

    public QueryBuilder<T> OrderBy<TKey>(Func<T, TKey> keySelector, bool descending = false,
                                         string fieldName = "")
    {
        string direction = descending ? "DESC" : "ASC";
        string clause = string.IsNullOrEmpty(fieldName) ? "?" : fieldName;
        _orderByClauses.Add($"{clause} {direction}");

        _source = descending ?
            _source.OrderByDescending(keySelector) :
            _source.OrderBy(keySelector);

        return this;
    }

    public QueryBuilder<T> Take(int count)
    {
        _takeCount = count;
        _source = _source.Take(count);
        return this;
    }

    public QueryBuilder<T> Skip(int count)
    {
        _skipCount = count;
        _source = _source.Skip(count);
        return this;
    }

    public QueryBuilder<TResult> Select<TResult>(Func<T, TResult> selector, string fields = "")
    {
        _selectClause = fields;
        return new QueryBuilder<TResult>(_source.Select(selector));
    }

    public QueryResult Execute()
    {
        var results = _source.ToList();

        string sql = BuildSqlString();

        return new QueryResult
        {
            SqlRepresentation = sql,
            Results = results,
            ResultCount = results.Count
        };
    }

    private string BuildSqlString()
    {
        var sb = new System.Text.StringBuilder();
        sb.AppendLine("SELECT " + (_selectClause ?? "*"));
        sb.AppendLine("FROM Source");

        if (_conditions.Any())
        {
            sb.AppendLine("WHERE " + string.Join(" AND ", _conditions));
        }

        if (_orderByClauses.Any())
        {
            sb.AppendLine("ORDER BY " + string.Join(", ", _orderByClauses));
        }

        if (_skipCount.HasValue)
        {
            sb.AppendLine($"OFFSET {_skipCount.Value} ROWS");
        }

        if (_takeCount.HasValue)
        {
            sb.AppendLine($"FETCH NEXT {_takeCount.Value} ROWS ONLY");
        }

        return sb.ToString();
    }
}

public class QueryResult
{
    public string SqlRepresentation { get; set; }
    public IList Results { get; set; }
    public int ResultCount { get; set; }

    public void Display()
    {
        Console.WriteLine("\n=== 生成的查询 ===");
        Console.WriteLine(SqlRepresentation);
        Console.WriteLine($"\n执行结果: {ResultCount} 条记录");

        if (Results.Count > 0 && Results.Count <= 10)
        {
            Console.WriteLine("\n数据预览:");
            foreach (var item in Results)
                Console.WriteLine($"  {item}");
        }
    }
}

// 使用示例
public class QueryBuilderDemo
{
    public void RunDemo()
    {
        var products = TestData.GetProducts();

        Console.WriteLine("=== 查询构建器演示 ===\n");

        // 查询1：查找促销的高价商品
        var query1 = new QueryBuilder<Product>(products)
            .Where(p => p.IsOnSale, "IsOnSale = true")
            .Where(p => p.Price > 3000, "Price > 3000")
            .OrderBy(p => p.Price, descending: true, fieldName: "Price")
            .Take(5)
            .Execute();

        query1.Display();

        // 查询2：查找库存不足的手机
        Console.WriteLine("\n" + new string('=', 50));
        var query2 = new QueryBuilder<Product>(products)
            .Where(p => p.Category == "手机", "Category = '手机'")
            .Where(p => p.Stock < 100, "Stock < 100")
            .OrderBy(p => p.Stock, fieldName: "Stock")
            .Select(p => new { p.Name, p.Stock, p.Price }, "Name, Stock, Price")
            .Execute();

        query2.Display();

        // 查询3：分页查询
        Console.WriteLine("\n" + new string('=', 50));
        var query3 = new QueryBuilder<Product>(products)
            .OrderBy(p => p.Name, fieldName: "Name")
            .Skip(2)
            .Take(3)
            .Execute();

        query3.Display();
    }
}

// 运行查询构建器演示
var builderDemo = new QueryBuilderDemo();
builderDemo.RunDemo();
```
</details>

---

## 八、总结

LINQ是C#中最强大和优雅的特性之一，掌握它将极大提升你的开发效率：

### 核心要点回顾

| 方法 | 用途 | SQL对应 |
|------|------|---------|
| `Where()` | 过滤筛选 | WHERE |
| `Select()` | 投影转换 | SELECT |
| `OrderBy()` | 排序 | ORDER BY |
| `First()` | 获取首个 | LIMIT 1 |
| `Any()` | 存在性判断 | EXISTS |
| `Count()` | 计数 | COUNT(*) |
| `Sum/Avg/Max/Min` | 聚合函数 | SUM/AVG/MAX/MIN |
| `GroupBy()` | 分组 | GROUP BY |
| `Take/Skip` | 分页 | LIMIT/OFFSET |

### 学习建议

1. **多用练**：在日常编码中主动使用LINQ替代传统的for循环
2. **读源码**：看开源项目中LINQ的使用方式
3. **理解原理**：了解延迟执行和IEnumerable/IQueryable的区别
4. **注意性能**：大数据集场景下要注意查询优化

> **下一步**：学习【异步编程初探】，了解如何编写高性能的非阻塞代码，这是现代Web开发的必备技能！

---
