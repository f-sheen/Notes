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

  

## 6、模式匹配 + 作用域提升

​	obj是object类型，无论实际存的是什么类型，编译器只知道它是 object。例如如果实际存的是List<T>类型，但是无法直接枚举
​	

```c#
if (x is T y)
{
    // 前提：在这个 {} 块内，x 的类型是 T
    //做了三件事，运行时检查-->（如果是目标类型）安全转换-->声明新变量y，类型为T 
    //y的作用域只存在内部
}
```

## 7、ASP.NET Core的启动

- ### 启动流程

  ```
  Program.Main()
  │
  ├─ CreateHostBuilder() → 配置主机（日志、配置、Startup）
  │
  ├─ host.Build()
  │   └─ 执行 Startup.ConfigureServices() → 注册所有服务 → 构建 DI 容器
  │
  └─ host.Run()
      └─ 执行 Startup.Configure() → 构建中间件管道
      └─ 启动 Kestrel → 开始处理 HTTP 请求
  ```

  1. 创建 HostBuilder，即CreateHostBuilder(args)：构建一个通用主机（Generic Host），包含 Web 功能（Kestrel、配置、日志等），他会创建一个 HostBuilder，自动注册基础服务，如：IConfiguration、IHostEnvironment、IHostApplicationLifetime、ILoggerFactory ，设置默认的配置源（如 appsettings.json、环境变量）

  2. ConfigureWebHostDefaults()： ASP.NET Core 为 Web 应用提供的默认配置模板，它在通用主机（Generic Host）基础上，自动添加 Web 所需的功能：Kestrel 服务器、默认配置源（如 appsettings.json）、默认日志提供程序、注册 IWebHostEnvironment、支持 HTTPS、Startup 类等

     - UseStartup<Startup>()：指定应用的启动类为 Startup（定义了 ConfigureServices 和 Configure）

     - CaptureStartupErrors：启用 Startup 错误捕获，如果 Startup.Configure 方法抛出异常（比如中间件配置错误），Kestrel 不会直接崩溃。而是返回一个 500 Internal Server Error 页面（配合下一行可显示详细错误）。默认值：开发环境为 true，生产环境为 false

     - UseSetting(WebHostDefaults.DetailedErrorsKey, "true")：启用 详细错误页面，当发生未处理异常时，浏览器会显示 完整的堆栈跟踪信息（而不仅是“An error occurred”）。⚠️ 安全提示：仅用于开发环境！ 生产环境应关闭，避免泄露敏感信息

     - UseShutdownTimeout：设置应用优雅关闭的超时时间，当收到关闭信号（如 Ctrl+C、K8s SIGTERM），Web 主机会等待最多 10 秒让正在处理的请求完成。超时后强制终止

     - ConfigureAppConfiguration：配置系统核心，加载 appsettings.json、环境变量、命令行参数等配置源

       ```
       //后加载的覆盖先加载的
       appsettings.json 
       ↓  
       appsettings.{Environment}.json  
       ↓  
       环境变量  
       ↓  
       命令行参数
       ```

       

     - ConfigureLogging：日志提供者配置

  3. host.Build()：注册 Startup.ConfigureServices 所有服务，构建最终的 IServiceProvider（DI 容器），准备中间件管道（但尚未执行 Configure），此时 DI 容器已就绪，可通过 host.Services 访问

  4. host.Run()：执行Startup.Configure()，配置 HTTP 请求管道（中间件），启动 Kestrel 服务器，监听端口，应用进入运行状态，可处理 HTTP 请求

  5. 处理请求：当HTTP请求到达时，依次经过中间件（日志、CORS、认证等）-->路由到Controller-->DI 容器为本次请求创建 新的 Scope-->Controller 及其依赖被解析-->执行业务逻辑

- ### ConfigureServices 

  ​	ConfigureServices(IServiceCollection services) 方法是整个应用程序依赖注入（DI）容器配置的核心入口点。它的主要职责是：向 DI 容器注册服务（Services），以便在应用生命周期中通过构造函数注入等方式使用。

  👽主要作用详解

  1. 注册应用程序所需的服务

     这是最核心的功能。你在这里告诉框架：

     - 哪些类需要被创建和管理

     - 它们的生命周期是怎样的（Singleton / Scoped / Transient）

     - 接口和实现之间的映射关系是什么

       ```c#
       public void ConfigureServices(IServiceCollection services)
       {
           // 注册自定义服务
           services.AddScoped<IUserService, UserService>();
       }
       ```

       

  2. 配置框架内置功能

     ASP.NET Core 的很多功能都是“可选中间件 + 服务注册”模式。你需要在这里“开启”它们，ConfigureServices 只负责“注册服务”，不负责“启用中间件”。中间件是在 Configure 方法中通过 app.UseXXX() 启用的

     <img src="E:\Notes\笔记图片\配置框架内置功能.png" style="zoom:25%;" />

  3. 读取配置（Configuration）

     你可以通过 IConfiguration（通常通过构造函数注入到 Startup）来读取 appsettings.json、环境变量等配置，并用于服务注册。

  4. 设置服务生命周期
     通过以下方法指定服务的生命周期：AddTransient、AddScoped、AddSingleton

  5. 集成第三方库或自定义扩展

     很多第三方库（如 AutoMapper、MediatR、FluentValidation）或你自己封装的模块，都会提供 IServiceCollection 扩展方法，在这里调用

     ```c#
     services.AddAutoMapper(typeof(Startup));
     services.AddMediatR(typeof(Startup));
     services.AddFluentValidation();
     ```

- ### Configure

## 8、生命周期

​	服务的生命周期（Service Lifetime） 决定了：服务实例何时被创建、被谁共享、何时被释放

1. Transient（瞬态）

   创建规则：

   - 每次请求服务时，都创建一个新实例
   - 即使在同一作用域（如同一个 HTTP 请求）内多次请求，也会得到不同实例
   - 用完就扔，次次新鲜

   应用场景：

   - 无状态、轻量级、线程安全的服务
   - 每次操作需要独立状态（如生成唯一 ID、临时计算）
   - 工具类（Utility）、映射器（Mapper）、工厂（Factory）

   注意事项：

   - 不适合持有昂贵资源（如数据库连接），因为频繁创建/销毁开销大
   - 如果注入到 Scoped 或 Singleton 服务中，会导致“捕获瞬态服务”问题（可能引发内存泄漏或状态错误）

2. Scoped（作用域）

   创建规则：

   - 每个作用域（Scope）内只创建一次实例

   - 在 ASP.NET Core 中，每个 HTTP 请求 = 一个作用域

     ​	同一请求内的所有 Scoped 服务共享同一个实例，不同请求之间实例彼此隔离

   - 一次请求，一个实例

   应用场景：

   - Entity Framework 的 DbContext：保证一个请求内使用同一个数据库上下文，支持事务和变更跟踪
   - 用户会话相关服务（如当前用户信息、租户上下文）
   - 需要在单次请求中保持状态一致性的服务

   注意事项：

   - 不能从 Singleton 服务中注入 Scoped 服务（会报错或导致 Scoped 实例被提升为“伪单例”）
   - 在非 Web 场景（如后台服务）中，需手动创建作用域

3. Singleton（单例）

   创建规则：

   - 整个应用程序生命周期内只创建一次实例
   - 所有请求、所有线程共享同一个对象
   - 全局唯一，小心并发

   应用场景：

   - 全局共享、无状态、线程安全的服务
   - 缓存管理器（如 MemoryCache）
   - 日志记录器（ILogger 通常是单例包装）
   - 配置提供者（IOptions）
   - 连接池、计数器、全局状态管理器（需线程安全！）

   注意事项：

   - 必须是线程安全的！ 多个请求并发访问同一个实例
   - 不能持有或依赖 Scoped / Transient 服务（会导致状态污染或内存泄漏）
   - 避免在单例中存储用户特定数据（会跨用户共享！）

   生命周期依赖关系
   服务可以依赖生命周期等于或更长的服务，但不能依赖更短的，Singleton ≥ Scoped ≥ Transient

## 9、一些问题

- **问题一：注册≠实例化**

  ​	对于services.AddScoped<T>(x => { ... });在应用启动时 不会执行 lambda 体（即 { ... } 中的代码），但 服务注册本身是成功的

  - *注册：*发生在 Startup.ConfigureServices 或 Program.cs 中，只是告诉 DI 容器：“如果有人要 IQuestDb，就用这个工厂方法创建”
  - *实例化：*发生在第一次有代码请求 IQuestDb 时（比如 Controller 构造函数注入），此时才会执行 sp => { ... }

  当某个类（如 QuestDbTestController）通过构造函数注入 IQuestDb 时

  ```c#
  public QuestDbTestController(IQuestDb questDb) // ← 这里触发
  ```

  DI 容器发现：“哦，需要 IQuestDb，我有一个注册的工厂方法！”
  于是 在这个 HTTP 请求的作用域内，第一次 调用你的 lambda，执行 { ... }，返回实例。
  同一个请求中后续再注入 IQuestDb，会复用同一个实例（Scoped 特性）。

  AddScoped 的生命周期是 “每个作用域一个实例”
  在 ASP.NET Core 中，每个 HTTP 请求 = 一个独立的作用域（scope）
  所以：第 1 个请求 → 执行 lambda 1 次；第 2 个请求 → 再执行 lambda 1 次；第 N 个请求 → 执行第 N 次

  

- **问题二：AddSingleton() 和 AddSingleton<T>(factory) 有什么区别？**

  - AddSingleton(instance)：直接注册一个已有实例
  - AddSingleton<T>(factory)：每次需要时调用工厂方法

- **问题三：映射关系**

  1. 注册服务

     ```c#
     var dbConfig = new ConnectionConfig { ... };
     services.AddSingleton(dbConfig);
     ```

     - 这行代码将一个具体的 ConnectionConfig 对象实例注册到 DI 容器中

     - 注册的服务类型（Service Type）是 ConnectionConfig

     - 内部相当于：services.AddSingleton<ConnectionConfig>(dbConfig);

     - 此时，DI 容器内部维护了一个映射表（简化表示）

       ```
       Service Type          → Instance
       ──────────────────────────────────────────────
       ConnectionConfig      → dbConfig (创建的对象)
       IHostedService        → DbInitializer (稍后注册)
       IQuestDb              → 工厂方法 (稍后注册)
       ```

  2. 托管服务

     ```c#
     services.AddHostedService<DbInitializer>();
     这行代码等价于：
     services.AddSingleton<IHostedService, DbInitializer>();
     ```

     - 它告诉容器：“DbInitializer 是一个 IHostedService，生命周期是 Singleton”

     - 当应用启动时，.NET 会尝试 构造 DbInitializer 的实例

     - 构造函数是：

       ```c#
       public DbInitializer(ConnectionConfig dbConfig)
       ```

       

     - 容器发现：“哦，它需要一个 ConnectionConfig！”

     - 于是去服务注册表中查找：有没有注册过 ConnectionConfig？

     - 有！ 就是你前面 AddSingleton(dbConfig) 注册的那个

     - 于是容器自动把 dbConfig 传入构造函数

- 问题四：
