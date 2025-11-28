## 1、IEnumerable<T> 接口

​	 .NET 中最基础的集合接口，提供遍历集合的能力，支持 foreach 循环和 LINQ 操作

特点：

- 只读遍历：只能读取元素，不能直接修改
- 延迟执行：LINQ 操作通常是延迟执行的
- 内存高效：不需要一次性加载所有数据到内存

```c#
// 1. 数组
IEnumerable<string> stations = new string[] { "Station1", "Station2", "Station3" };

// 2. List<T>
IEnumerable<string> stations = new List<string> { "Station1", "Station2", "Station3" };

// 3. 使用 LINQ 查询结果
IEnumerable<string> stations = from station in allStations 
                              where station.Contains("OP") 
                              select station;
// 例子
// 延迟执行 - 只有在遍历时才执行
IEnumerable<string> result = stations.Where(s => s.Contains("OP"));
// 此时还没有真正执行查询

foreach (var item in result)  // 此时才执行查询
{
    Console.WriteLine(item);
}
// 立即执行 - 查询立即完成
List<string> result = stations.Where(s => s.Contains("OP")).ToList();
// 查询已经执行完毕，结果存储在内存中
```

## 2、ORM的限制

- 无法在 LINQ 查询中调用任意方法

- 无法访问复杂的静态属性

- 只能处理简单的表达式和局部变量引用

  解决方案的核心：在表达式树构建之前，提前获取属性的值，让 SqlSugar 能够直接使用具体的值而不是属性引用。

  举例：

  ```c#
  //LINQ to Objects（内存中）
  // ✅ 可以这样写，因为是在内存中操作
  .GroupBy(ws => ws.OpName)
  .Select(g => g.OrderByDescending(ws => ws.UpdateTime).First())
  
  //SqlSugar
  // ❌ 不能这样写，因为无法翻译成 SQL
  .GroupBy(ws => ws.OpName)
  .Select(g => g.OrderByDescending(ws => ws.UpdateTime).First())
  ```

  原因：

  - SQL 限制：SQL 的 GROUP BY 子句只能选择分组列和聚合函数

  - 翻译问题：SqlSugar 需要将 LINQ 翻译成 SQL，而 g.OrderByDescending().First() 无法直接翻译

  - 语义差异：数据库中的分组操作与内存中的分组操作本质不同

    

## 3、数据库和内存的使用领域

核心原则：让数据库做它擅长的事情（筛选、排序、聚合），让内存做复杂的数据处理。

### 	👍应该在数据库中实现的操作

1. 数据筛选和过滤（Where）

2. 排序和分页（OrderBy，Skip，Take）

3. 聚合计算（SqlFunc）

4. 表连接（Join）

5. 去重操作（Distinct）

   

### 	👍应该在内存中实现的操作

1. 复杂的分组后处理
2. 复杂的数据转换
3. 递归操作或树形结构处理
4. 多次使用相同数据的复杂业务逻辑
5. 需要调用C#特定API的操作

## 4、DateTime和TimeSpan

### DateTime：表示一个特定的时间点，包含日期和时间信息

应用场景

1. 记录事件发生时间
2. 日期计算和比较
3. 日程安排

```c#
//构造函数
DateTime date1 = new DateTime(2024, 1, 15);           // 年月日
DateTime date2 = new DateTime(2024, 1, 15, 10, 30, 0); // 年月日时分秒
DateTime date3 = new DateTime(2024, 1, 15, 10, 30, 0, 500); // 年月日时分秒毫秒

//静态属性
DateTime now = DateTime.Now;        // 当前本地时间
DateTime utcNow = DateTime.UtcNow;  // 当前UTC时间
DateTime today = DateTime.Today;    // 今天日期（时间00:00:00）
DateTime minValue = DateTime.MinValue; // 最小日期值
DateTime maxValue = DateTime.MaxValue; // 最大日期值

//实例属性
DateTime dt = new DateTime(2024, 1, 15, 10, 30, 45);
int year = dt.Year;          // 2024
int month = dt.Month;        // 1
int day = dt.Day;            // 15
int hour = dt.Hour;          // 10
int minute = dt.Minute;      // 30
int second = dt.Second;      // 45
int millisecond = dt.Millisecond; // 0
DayOfWeek dayOfWeek = dt.DayOfWeek; // Monday
int dayOfYear = dt.DayOfYear;       // 15
DateTime.Date = dt.Date;            // 日期部分（时间00:00:00）

//计算方法
DateTime dt = new DateTime(2024, 1, 15);
DateTime tomorrow = dt.AddDays(1);//添加时间
DateTime nextHour = dt.AddHours(1);
DateTime nextMonth = dt.AddMonths(1);
DateTime nextYear = dt.AddYears(1);
TimeSpan span = TimeSpan.FromHours(2);// 添加TimeSpan
DateTime future = dt.Add(span);
DateTime yesterday = dt.AddDays(-1);// 减法
TimeSpan difference = dt.Subtract(DateTime.Now);
```



### TimeSpan：表示一个时间间隔或持续时间，不关联特定的起始点

应用场景

1. 计算时间差
2. 定时器和倒计时

```c#
//构造函数
TimeSpan ts1 = new TimeSpan(10, 30, 0);        // 10小时30分钟
TimeSpan ts2 = new TimeSpan(1, 10, 30, 45);    // 1天10小时30分钟45秒
TimeSpan ts3 = new TimeSpan(1, 10, 30, 45, 500); // 1天10小时30分钟45秒500毫秒

//静态工厂方法
TimeSpan fromDays = TimeSpan.FromDays(1.5);        // 1天12小时
TimeSpan fromHours = TimeSpan.FromHours(2.5);      // 2小时30分钟
TimeSpan fromMinutes = TimeSpan.FromMinutes(90);   // 1小时30分钟
TimeSpan fromSeconds = TimeSpan.FromSeconds(125);  // 2分钟5秒
TimeSpan fromMilliseconds = TimeSpan.FromMilliseconds(1500); // 1.5秒
TimeSpan zero = TimeSpan.Zero;                     // 零时间间隔
TimeSpan maxValue = TimeSpan.MaxValue;             // 最大时间间隔
TimeSpan minValue = TimeSpan.MinValue;             // 最小时间间隔
TimeSpan ts = new TimeSpan(1, 10, 30, 45, 500);//实例属性
int days = ts.Days;              // 1
int hours = ts.Hours;            // 10
int minutes = ts.Minutes;        // 30
int seconds = ts.Seconds;        // 45
int milliseconds = ts.Milliseconds; // 500
double totalMilliseconds = ts.TotalMilliseconds; // 约124245500毫秒
TimeSpan ts1 = TimeSpan.FromHours(2);//计算方法
TimeSpan ts2 = TimeSpan.FromMinutes(30);
TimeSpan sum = ts1.Add(ts2);                    // 加法：2小时30分钟
TimeSpan added = ts1 + ts2;                     // 同上
TimeSpan difference = ts1.Subtract(ts2);        // 减法：1小时30分钟
TimeSpan subtracted = ts1 - ts2;                // 同上
TimeSpan doubled = ts1.Multiply(2);             //乘法： 4小时
TimeSpan doubledOp = ts1 * 2;                   // 同上
TimeSpan halved = ts1.Divide(2);                // 除法：小时
TimeSpan halvedOp = ts1 / 2;                    // 同上
TimeSpan ts1 = TimeSpan.FromHours(2);//比较方法
TimeSpan ts2 = TimeSpan.FromHours(1);
bool isEqual = ts1.Equals(ts2);                 // false
int compare = ts1.CompareTo(ts2);               // 1 (大于)
bool isGreater = ts1 > ts2;                     // true
```

## 5、项目引用、Using和DependsOn

​	项目引用是Using和DependsOn基础，没有项目引用就无法访问对方的代码

- 项目引用：我能"看到"谁的代码（物理依赖）

- Using：我怎么写代码更省事（语法便捷）

- DependsOn：谁要先启动我才能启动（运行时初始化顺序），实际作用是 控制模块初始化顺序，确保被依赖的模块先初始化，当前模块后初始化

  ```c#
  // 1. 数据库模块 - 最底层
  public class DataModule : AbpModule
  {
      public override void ConfigureServices(ServiceConfigurationContext context)
      {
          context.Services.AddDbContext<AppDbContext>();
          Console.WriteLine("DataModule 初始化");
      }
  }
  
  // 2. 商品模块 - 依赖数据模块
  [DependsOn(typeof(DataModule))]
  public class ProductModule : AbpModule
  {
      public override void ConfigureServices(ServiceConfigurationContext context)
      {
          // 这里可以安全使用DataModule注册的DbContext
          context.Services.AddTransient<IProductService, ProductService>();
          Console.WriteLine("ProductModule 初始化");
      }
  }
  
  // 3. 订单模块 - 依赖商品模块和数据模块
  [DependsOn(typeof(ProductModule))]
  public class OrderModule : AbpModule
  {
      public override void ConfigureServices(ServiceConfigurationContext context)
      {
          // 这里可以安全使用ProductModule注册的IProductService
          context.Services.AddTransient<IOrderService, OrderService>();
          Console.WriteLine("OrderModule 初始化");
      }
  }
  
  // 4. Web API模块 - 最上层，依赖所有业务模块
  [DependsOn(typeof(OrderModule), typeof(ProductModule))]
  public class WebModule : AbpModule
  {
      public override void ConfigureServices(ServiceConfigurationContext context)
      {
          context.Services.AddControllers();
          Console.WriteLine("WebModule 初始化");
      }
  }
  ```

  

## 6、