
---
title: C#（一） 委托与事件实战示例：深入解析委托、Action 及定时器的实现与应用
date: 2026-05-24 08:00:00
tags:
  - "C#"
  - ".NET"
  - "委托"
  - "事件"
categories:
  - "编程语言"
---

# Program1

```csharp
using System;

namespace HelloWord
{
    class Program
    {
        public class Staff
        {
            public Staff()
            {
                Console.WriteLine("员工类初始化");
            }

            public int Number { get; set; }
            public Staff(int number)
            {
                Number = number;
            }
        }

        public class Manager : Staff
        {
            public Manager()
            {
                Console.WriteLine("经理类初始化");
            }
            public Manager(int number)
            {
            }
        }

        static void Main(string[] args)
        {
            var manager = new Manager(123);
            Console.WriteLine(manager.Number);

            Console.Read();
        }
    }
}

```

## 解释
### 1. 引用命名空间

```csharp
using System;
```

- `using` 关键字用于引入命名空间。`System` 是 .NET 框架中最基础的命名空间，它包含了许多常用的类型和功能，比如 `Console` 类，用于控制台输入输出操作。

### 2. 命名空间定义

```csharp
namespace HelloWord
{
    // ...
}
```

- `namespace` 用于组织代码，将相关的类和类型分组在一起，避免命名冲突。这里定义了一个名为 `HelloWord` 的命名空间，后续的类都包含在这个命名空间中。

### 3. 类定义

#### `Program` 类

```csharp
class Program
{
    // ...
}
```

- `Program` 类是程序的入口点类，在 C# 控制台应用程序中，`Main` 方法必须定义在一个类中，通常这个类就命名为 `Program`。

#### `Staff` 类

```csharp
public class Staff
{
    public Staff()
    {
        Console.WriteLine("员工类初始化");
    }

    public int Number { get; set; }
    public Staff(int number)
    {
        Number = number;
    }
}
```



- ```Staff  ```类是一个公共类，代表员工。

  - **构造函数 `Staff()`**：这是一个无参构造函数，当创建 `Staff` 类的对象时，如果不传递参数，就会调用这个构造函数，它会在控制台输出 "员工类初始化"。
  - **属性 `Number`**：这是一个自动属性，使用 `get` 和 `set` 访问器，允许外部代码获取和设置 `Number` 属性的值。
  - **构造函数 `Staff(int number)`**：这是一个带参数的构造函数，接受一个整数参数 `number`，并将其赋值给 `Number` 属性。

#### `Manager` 类

```csharp
public class Manager : Staff
{
    public Manager()
    {
        Console.WriteLine("经理类初始化");
    }
    public Manager(int number)
    {
    }
}
```



- ```Manager  ```类是一个公共类，它继承自 ```Staff  ```类，这意味着 ```Manager  ```类拥有 ```Staff  ```类的所有公共成员。

  - **构造函数 `Manager()`**：这是一个无参构造函数，当创建 `Manager` 类的对象时，如果不传递参数，就会调用这个构造函数，它会在控制台输出 "经理类初始化"。
  - **构造函数 `Manager(int number)`**：这是一个带参数的构造函数，接受一个整数参数 `number`，但在这个构造函数中没有任何代码，所以不会对传入的参数进行任何处理。

### 4. `Main` 方法

```csharp
static void Main(string[] args)
{
    var manager = new Manager(123);
    Console.WriteLine(manager.Number);

    Console.Read();
}
```



- ```Main  ```方法是程序的入口点，程序从这里开始执行。

  - **创建 `Manager` 对象**：`var manager = new Manager(123);` 这行代码创建了一个 `Manager` 类的对象，并传递参数 `123`。由于 `Manager` 类的带参构造函数没有调用基类 `Staff` 的带参构造函数，所以 `Staff` 类的无参构造函数会被自动调用，在控制台输出 "员工类初始化"。然后 `Manager` 类的带参构造函数执行，由于其内部没有代码，不会有其他输出。
  - **输出 `Number` 属性的值**：`Console.WriteLine(manager.Number);` 这行代码输出 `manager` 对象的 `Number` 属性的值。由于 `Manager` 类的带参构造函数没有对 `Number` 属性进行赋值，而 `Staff` 类的无参构造函数也没有对 `Number` 属性进行初始化，所以 `Number` 属性的默认值为 `0`，因此控制台会输出 `0`。
  - **等待用户输入**：`Console.Read();` 这行代码会暂停程序的执行，等待用户输入一个字符，直到用户按下任意键后程序才会退出。
# Program2

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

namespace HelloWord
{
    class Program
    {
        public class Shape
        {
            public int Width { get; set; }
            public int Height { get; set; }
            public int X { get; set; }
            public int Y { get; set; }
            public void Draw()
            {
                Console.WriteLine($"Width: {Width}, Height: {Height}, position: ({X}, {Y})");
            }
        }

        public class Text : Shape
        {
            public int FontSize { get; set; }
            public string FontName { get; set; }
        }

        static void Main(string[] args)
        {
            var text = new Text();
            Shape shape = text;

            string[] cars = { "Volvo", "BMW", "Ford", "Mazda" };
            //cars[0] = 1;

            var arrays = new ArrayList();
            arrays.Add(1);
            arrays.Add("123");
            arrays.Add(text);
            arrays.Add(new Shape());
            arrays.Add(cars);

            var shapeList = new List<Shape>();
            shapeList.Add(text);
            shapeList.Add(new Shape());

            // 向下引用 
            var shape0 = shapeList[0];
            var textList = new List<Text>();
            shapeList.ForEach(s =>
            {
                if (s is Text)
                {
                    textList.Add((Text)s);
                }
            });



            Console.Read();
        }
    }
}
```
## 解释
### 1. 命名空间和引用

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
```

- ```using  ```关键字用于引入命名空间，这样在代码中就可以直接使用这些命名空间下的类型，而无需使用完整的命名空间限定。

  - `System`：包含了许多基础类型和常用功能，如 `Console` 类，用于控制台输入输出。
  - `System.Collections`：提供了非泛型集合类，如 `ArrayList`，可以存储不同类型的对象。
  - `System.Collections.Generic`：提供了泛型集合类，如 `List<T>`，可以存储特定类型的对象。

### 2. 命名空间定义

```csharp
namespace HelloWord
{
    // ...
}
```

- `namespace` 用于组织代码，将相关的类型放在一个命名空间中，避免命名冲突。这里定义了一个名为 `HelloWord` 的命名空间。

### 3. 类定义

#### `Shape` 类

```csharp
public class Shape
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int X { get; set; }
    public int Y { get; set; }
    public void Draw()
    {
        Console.WriteLine($"Width: {Width}, Height: {Height}, position: ({X}, {Y})");
    }
}
```



- ```Shape  ```是一个公共类，代表一个形状。

  - `Width`、`Height`、`X` 和 `Y` 是属性，使用自动属性语法（`{ get; set; }`），允许对这些属性进行读写操作。
  - `Draw` 方法用于在控制台输出形状的宽度、高度和位置信息。

#### `Text` 类

```csharp
public class Text : Shape
{
    public int FontSize { get; set; }
    public string FontName { get; set; }
}
```



- `Text` 类继承自 `Shape` 类，这意味着 `Text` 类拥有 `Shape` 类的所有属性和方法，同时还添加了自己的属性 `FontSize` 和 `FontName`，分别表示字体大小和字体名称。

### 4. `Main` 方法

#### 创建 `Text` 对象并进行类型转换

```csharp
var text = new Text();
Shape shape = text;
```

- `var` 是隐式类型声明，编译器会根据右侧的表达式推断变量的类型。这里创建了一个 `Text` 对象 `text`。
- 将 `text` 赋值给 `Shape` 类型的变量 `shape`，这是一个向上转型，因为 `Text` 是 `Shape` 的子类，这种转换是安全的。

#### 创建数组

```csharp
string[] cars = { "Volvo", "BMW", "Ford", "Mazda" };
//cars[0] = 1;
```

- 创建了一个字符串数组 `cars`，并初始化了四个字符串元素。
- 注释掉的代码 `cars[0] = 1;` 是错误的，因为数组 `cars` 被声明为 `string` 类型，不能将整数赋值给字符串数组的元素。

#### 使用 `ArrayList`

```csharp
var arrays = new ArrayList();
arrays.Add(1);
arrays.Add("123");
arrays.Add(text);
arrays.Add(new Shape());
arrays.Add(cars);
```

- `ArrayList` 是一个非泛型集合，可以存储不同类型的对象。
- 依次向 `ArrayList` 中添加了一个整数、一个字符串、一个 `Text` 对象、一个 `Shape` 对象和一个字符串数组。

#### 使用泛型 `List<Shape>`

```csharp
var shapeList = new List<Shape>();
shapeList.Add(text);
shapeList.Add(new Shape());
```

- `List<Shape>` 是一个泛型集合，只能存储 `Shape` 类型或其子类型的对象。
- 向 `shapeList` 中添加了一个 `Text` 对象和一个 `Shape` 对象。

#### 向下转型

```csharp
var shape0 = shapeList[0];
var textList = new List<Text>();
shapeList.ForEach(s =>
{
    if (s is Text)
    {
        textList.Add((Text)s);
    }
});
```

- `shape0` 是从 `shapeList` 中获取的第一个元素，类型为 `Shape`。
- 创建了一个 `List<Text>` 类型的 `textList`，用于存储 `Text` 对象。
- 使用 `ForEach` 方法遍历 `shapeList`，对于每个元素，使用 `is` 关键字检查它是否为 `Text` 类型，如果是，则将其强制转换为 `Text` 类型并添加到 `textList` 中。

#### 等待用户输入

```csharp
Console.Read();
```

- `Console.Read()` 方法用于读取用户从控制台输入的一个字符，并等待用户输入。这可以防止控制台窗口在程序执行完毕后立即关闭。

# Program3：依赖注入

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;

namespace HelloWord
{

    class Program
    {
        static void Main(string[] args)
        {
            // 创建订单
            var order = new Order
            {
                Id = 123,
                DatePlaced = DateTime.Now,
                TotalPrice = 30f
            };

            //配置 IOC 反转控制容器
            var collection = new ServiceCollection();
            collection.AddScoped<IShippingCalculator, DoubleElevenShippingCalculator>();
            collection.AddScoped<IOrderProcessor, OrderProcessor>();
            //collection.AddSingleton<IOrderProcessor, OrderProcessor>();
            //collection.AddTransient<IOrderProcessor, OrderProcessor>();
            IServiceProvider serviceProvider = collection.BuildServiceProvider();

            // 通过接口，从IOC中找出订单处理器
            var orderProcessor = serviceProvider.GetService<IOrderProcessor>();
            //var orderProcessor2 = serviceProvider.GetService<IOrderProcessor>();

            // 处理订单
            orderProcessor.Process(order);

            Console.Read();
        }
    }
}
```
## Order.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace HelloWord
{
    public class Order
    {
        public int Id { get; set; }
        public DateTime DatePlaced { get; set; }
        public Shipment Shipment { get; set; }
        public float TotalPrice { get; set; }
        public bool IsShipped { get; set; }
    }
}
```
## Process.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace HelloWord
{
    public class OrderProcessor : IOrderProcessor
    {
        private readonly IShippingCalculator _shippingCalculator;

        public OrderProcessor(IShippingCalculator shippingCalculator)
        {
            Console.WriteLine("OrderProcessor被构造了一次");
            _shippingCalculator = shippingCalculator;
        }

        public void Process(Order order)
        {
            if (order.IsShipped)
                throw new InvalidOperationException("订单已发货，无法重复处理");

            order.Shipment = new Shipment
            {
                Cost = _shippingCalculator.CalculateShipping(order),
                ShippingDate = DateTime.Today.AddDays(1)
            };
            order.IsShipped = true;
            Console.WriteLine($"订单#{order.Id}完成，已发货");
        }
    }
}
```

## 解释

### 1. 引入命名空间
```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
```
- `System`：这是 .NET 中最基础的命名空间，包含了许多常用的类型和功能，例如 `DateTime` 类型等。
- `Microsoft.Extensions.DependencyInjection`：该命名空间提供了依赖注入（Dependency Injection, DI）相关的类和接口，用于实现控制反转（Inversion of Control, IOC）模式，像 `ServiceCollection` 和 `IServiceProvider` 都在这个命名空间下。

### 2. 定义命名空间和类
```csharp
namespace HelloWord
{
    class Program
    {
        static void Main(string[] args)
        {
            // ...
        }
    }
}
```
- `namespace HelloWord`：定义了一个名为 `HelloWord` 的命名空间，用于组织代码。
- `class Program`：定义了一个名为 `Program` 的类，其中包含了 `Main` 方法，它是程序的入口点。`static void Main(string[] args)` 是控制台应用程序的标准入口方法。

### 3. 创建订单对象
```csharp
var order = new Order
{
    Id = 123,
    DatePlaced = DateTime.Now,
    TotalPrice = 30f
};
```
- 这里创建了一个 `Order` 类的实例。
- `Id` 属性被设置为 `123`，表示订单的编号。
- `DatePlaced` 属性被设置为当前时间，记录订单的下单时间。
- `TotalPrice` 属性被设置为 `30f`，表示订单的总价。

### 4. 配置依赖注入容器
```csharp
var collection = new ServiceCollection();
collection.AddScoped<IShippingCalculator, DoubleElevenShippingCalculator>();
collection.AddScoped<IOrderProcessor, OrderProcessor>();
//collection.AddSingleton<IOrderProcessor, OrderProcessor>();
//collection.AddTransient<IOrderProcessor, OrderProcessor>();
IServiceProvider serviceProvider = collection.BuildServiceProvider();
```
- `var collection = new ServiceCollection();`：创建了一个 `ServiceCollection` 实例，它是一个用于注册服务的容器，我们可以向其中添加不同的服务。
- `collection.AddScoped<IShippingCalculator, DoubleElevenShippingCalculator>();`：使用 `AddScoped` 方法将 `IShippingCalculator` 接口注册为 `DoubleElevenShippingCalculator` 类的实现，并且服务的生命周期为 `Scoped`。这意味着在同一个请求作用域内，每次获取 `IShippingCalculator` 服务时都会返回同一个 `DoubleElevenShippingCalculator` 实例。
- `collection.AddScoped<IOrderProcessor, OrderProcessor>();`：同样使用 `AddScoped` 方法将 `IOrderProcessor` 接口注册为 `OrderProcessor` 类的实现，生命周期也是 `Scoped`。
- `collection.AddSingleton<IOrderProcessor, OrderProcessor>();` 和 `collection.AddTransient<IOrderProcessor, OrderProcessor>();` 是被注释掉的代码。`AddSingleton` 方法会将服务注册为单例模式，即在整个应用程序生命周期内只创建一个 `OrderProcessor` 实例；`AddTransient` 方法会将服务注册为瞬态模式，每次请求 `OrderProcessor` 服务时都会创建一个新的实例。
- `IServiceProvider serviceProvider = collection.BuildServiceProvider();`：调用 `BuildServiceProvider` 方法将 `ServiceCollection` 中注册的服务构建成一个 `IServiceProvider` 实例，这个实例可以用于解析和获取注册的服务。

### 5. 从依赖注入容器中获取服务实例
```csharp
var orderProcessor = serviceProvider.GetService<IOrderProcessor>();
//var orderProcessor2 = serviceProvider.GetService<IOrderProcessor>();
```
- `var orderProcessor = serviceProvider.GetService<IOrderProcessor>();`：通过 `IServiceProvider` 的 `GetService` 方法，根据 `IOrderProcessor` 接口从服务容器中获取对应的服务实例。由于之前将 `IOrderProcessor` 注册为 `OrderProcessor` 类的实现，所以这里获取到的是 `OrderProcessor` 类的实例。
- `var orderProcessor2 = serviceProvider.GetService<IOrderProcessor>();` 是被注释掉的代码，它的作用是再次从服务容器中获取 `IOrderProcessor` 服务的实例。在 `Scoped` 生命周期下，如果是在同一个请求作用域内，`orderProcessor` 和 `orderProcessor2` 将是同一个实例。

### 6. 处理订单
```csharp
orderProcessor.Process(order);
```
- 调用 `orderProcessor` 的 `Process` 方法来处理之前创建的订单对象 `order`。`Process` 方法根据 `IShippingCalculator` 计算运费，并设置订单的发货信息，最后标记订单为已发货。

### 7. 等待用户输入
```csharp
Console.Read();
```
- 调用 `Console.Read()` 方法等待用户输入，防止控制台窗口在程序执行完后立即关闭，方便我们查看程序的输出结果。


# 4.Program.cs

```csharp
using System;

namespace HelloWord
{
    struct Game
    {
        public string name;
        public string developer;
        public DateTime releaseDate;

        public Game(string name, string developer, DateTime releaseDate)
        {
            this.name = name;
            this.developer = developer;
            this.releaseDate = releaseDate;
        }

        public void GetInfo()
        {
            Console.WriteLine("游戏名称", name);
            Console.WriteLine("开发", developer);
            Console.WriteLine("上架日期", releaseDate);
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            Game game;



            game.name = "pokemon";
            game.developer = "Alex";
            game.releaseDate = DateTime.Today;

            game.GetInfo();

            Console.Read();
        }
    }
}
```
## 1. this 关键字的含义
this 关键字在类或者结构体的实例方法、构造函数中使用，它代表当前对象（也就是调用该方法或构造函数的那个对象实例）的引用。借助 this，你可以明确地引用当前对象的成员（像字段、属性、方法等）。
## 2. 使用 this 的必要性
在构造函数或者方法中，当参数名和类 / 结构体的成员名相同时，就需要使用 this 来区分局部参数和成员。要是不使用 this，编译器会把变量名解析为局部参数，而非对象的成员。

## 3. 结构体概述
在 C# 中，结构体（`struct`）是一种值类型，通常用于封装小型相关变量组。与类不同，结构体存储在栈上（除非是作为类的成员或引用类型的一部分），这使得结构体在某些场景下（如简单数据存储和传递）具有更高的性能。

### 3.1 结构体定义
```csharp
struct Game
{
    public string name;
    public string developer;
    public DateTime releaseDate;
    // ...
}
```
### 3.2 构造函数
```csharp
public Game(string name, string developer, DateTime releaseDate)
{
    this.name = name;
    this.developer = developer;
    this.releaseDate = releaseDate;
}
```
- **作用**：构造函数用于初始化结构体实例的成员变量。当创建 `Game` 结构体的新实例时，可以通过传递相应的参数来设置 `name`、`developer` 和 `releaseDate` 的值。
- **`this` 关键字**：在构造函数中，`this` 关键字用于区分局部参数和结构体的成员变量。`this.name` 指的是结构体的 `name` 成员，而 `name` 是构造函数的参数。
### 3.3. 结构体和类的区别
- **存储方式**：结构体是值类型，存储在栈上；类是引用类型，存储在堆上。
- **默认值**：结构体的实例有默认值，即使没有显式初始化成员变量，它们也会被赋予默认值；类的实例必须使用 `new` 关键字创建，否则会产生编译错误。
- **继承**：结构体不支持继承，不能作为基类或派生类；类支持继承，可以创建类的层次结构。

# 5.Program.cs：枚举

```csharp
using System;

namespace HelloWord
{
    class Program
    {
        enum Weekday
        {
            MONDAY = 1,
            TUESDAY,
            WEDNESDAY,
            THURSDAY,
            FRIDAY = 5,
            SATURDAY,
            SUNDAY
        }

        static void Main(string[] args)
        {
            Weekday firday = Weekday.FRIDAY;

            Console.WriteLine(firday);
            Console.WriteLine((int)firday);

            var firday2 = (Weekday)4;
            Console.WriteLine(firday2);

            Weekday day = Weekday.MONDAY;

            switch(day)
            {
                case Weekday.MONDAY:
                case Weekday.TUESDAY:
                case Weekday.WEDNESDAY:
                case Weekday.THURSDAY:
                case Weekday.FRIDAY:
                    Console.WriteLine("今天要上班");
                    break;
                case Weekday.SUNDAY:
                case Weekday.SATURDAY:
                    Console.WriteLine("家里蹲");
                    break;
            }

            Console.Read();
        }
    }
}
```
## 解释

### 1 枚举类型的定义
```csharp
enum Weekday
{
    MONDAY = 1,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY = 5,
    SATURDAY,
    SUNDAY
}
```
- `enum Weekday`：定义了一个名为 `Weekday` 的枚举类型，用于表示一周的七天。
- `MONDAY = 1`：显式地将 `MONDAY` 的值设置为 `1`。
- `TUESDAY`、`WEDNESDAY`、`THURSDAY`：没有显式赋值，它们的值会依次递增，即 `TUESDAY` 为 `2`，`WEDNESDAY` 为 `3`，`THURSDAY` 为 `4`。
- `FRIDAY = 5`：显式地将 `FRIDAY` 的值设置为 `5`。
- `SATURDAY`、`SUNDAY`：没有显式赋值，它们的值会依次递增，即 `SATURDAY` 为 `6`，`SUNDAY` 为 `7`。
### 2 枚举值的赋值和输出
```csharp
Weekday firday = Weekday.FRIDAY;

Console.WriteLine(firday);
Console.WriteLine((int)firday);
```
- `Weekday firday = Weekday.FRIDAY;`：定义一个 `Weekday` 类型的变量 `firday`，并将其赋值为 `Weekday.FRIDAY`。
- `Console.WriteLine(firday);`：输出枚举值的名称，即 `FRIDAY`。
- `Console.WriteLine((int)firday);`：将枚举值强制转换为整数类型并输出，即 `5`。

### 3 枚举值的类型转换
```csharp
var firday2 = (Weekday)4;
Console.WriteLine(firday2);
```
- `var firday2 = (Weekday)4;`：将整数 `4` 强制转换为 `Weekday` 枚举类型，并赋值给变量 `firday2`。由于 `4` 对应的枚举值是 `THURSDAY`，所以 `firday2` 的值为 `THURSDAY`。
- `Console.WriteLine(firday2);`：输出枚举值的名称，即 `THURSDAY`。

### 4 `switch` 语句
```csharp
Weekday day = Weekday.MONDAY;

switch(day)
{
    case Weekday.MONDAY:
    case Weekday.TUESDAY:
    case Weekday.WEDNESDAY:
    case Weekday.THURSDAY:
    case Weekday.FRIDAY:
        Console.WriteLine("今天要上班");
        break;
    case Weekday.SUNDAY:
    case Weekday.SATURDAY:
        Console.WriteLine("家里蹲");
        break;
}
```
- `Weekday day = Weekday.MONDAY;`：定义一个 `Weekday` 类型的变量 `day`，并将其赋值为 `Weekday.MONDAY`。
- `switch(day)`：根据 `day` 的值执行不同的操作。
- `case Weekday.MONDAY:` 到 `case Weekday.FRIDAY:`：如果 `day` 的值为 `MONDAY`、`TUESDAY`、`WEDNESDAY`、`THURSDAY` 或 `FRIDAY`，则输出 `今天要上班`，并使用 `break` 语句跳出 `switch` 语句。
- `case Weekday.SUNDAY:` 到 `case Weekday.SATURDAY:`：如果 `day` 的值为 `SUNDAY` 或 `SATURDAY`，则输出 `家里蹲`，并使用 `break` 语句跳出 `switch` 语句。

# Helloworld

```csharp
结构
-- Book.cs
-- DiscountCalculator.cs
-- List.cs
-- Nullable.cs
-- Product.cs
-- ProductList.cs
-- Program.cs
```
## 解释
### 1. `List.cs`
```csharp
using System;

namespace HelloWord
{
    public class List
    {
        public void Add(int number)
        {
            throw new NotImplementedException();
        }

        public int this[int index]
        {
            get { throw new NotImplementedException(); }
        }
    }
}
```
- **功能概述**：定义了一个名为`List`的类，该类有两个成员：一个`Add`方法用于添加整数元素，一个索引器用于通过索引访问列表中的元素。不过，这两个成员的具体实现都未完成，只是抛出了`NotImplementedException`异常。
#### 索引器
索引器允许类或结构的实例像数组一样被索引访问。
1. 索引器的返回类型
索引器的返回类型可以是任意有效的 C# 类型，不局限于 int。例如，如果要创建一个存储字符串的列表，索引器的返回类型可以是 string：

```csharp
using System;

namespace HelloWord
{
    public class StringList
    {
        private string[] strings = new string[10];

        public string this[int index]
        {
            get
            {
                if (index < 0 || index >= strings.Length)
                {
                    throw new IndexOutOfRangeException();
                }
                return strings[index];
            }
            set
            {
                if (index < 0 || index >= strings.Length)
                {
                    throw new IndexOutOfRangeException();
                }
                strings[index] = value;
            }
        }
    }
}
```
2. 索引参数的类型和数量
索引参数的类型也可以多种多样，除了 int，还可以是 string、double 等，甚至可以有多个索引参数。例如，一个二维数组的模拟类可以使用两个 int 作为索引参数：

```csharp
using System;

namespace HelloWord
{
    public class TwoDimensionalArray
    {
        private int[,] array = new int[10, 10];

        public int this[int row, int col]
        {
            get
            {
                if (row < 0 || row >= 10 || col < 0 || col >= 10)
                {
                    throw new IndexOutOfRangeException();
                }
                return array[row, col];
            }
            set
            {
                if (row < 0 || row >= 10 || col < 0 || col >= 10)
                {
                    throw new IndexOutOfRangeException();
                }
                array[row, col] = value;
            }
        }
    }
}
```

### 2. `Program.cs`
```csharp
using System;

namespace HelloWord
{
    class Program
    {
        static void Main(string[] args)
        {
            //var numberlist = new List();
            var numberlist = new GenericList<int>();
            numberlist.Add(1);
            numberlist.Add(2);

            //var productList = new ProductList();
            var productList = new GenericList<Product>();
            productList.Add(new Product()
            {
                Id = 1, 
                Price = 100
            });
            productList.Add(new Product()
            {
                Id = 2,
                Price = 200
            });
            
            var dic = new System.Collections.Generic.Dictionary<string, Product>();
            dic.Add("123", new Product());
            dic.Add("234", new Product());
            dic.Add("345", new Product());

            Product product;
            dic.TryGetValue("123", out product);

            Nullable<int> number = new Nullable<int>();
            Console.WriteLine(number.HasValue);

            var number2 = new Nullable<int>(5);
            number2.GetValueOrDefault();

            Console.Read();
        }
    }
}
```
- **功能概述**：这是程序的入口点。在`Main`方法中：
  - 创建了一个`GenericList<int>`类型的`numberlist`，并添加了两个整数元素。
  - 创建了一个`GenericList<Product>`类型的`productList`，并添加了两个`Product`对象。
  - 创建了一个`System.Collections.Generic.Dictionary<string, Product>`类型的`dic`，并添加了三个键值对。然后尝试通过键`"123"`获取对应的`Product`对象。
  - 创建了一个`Nullable<int>`类型的`number`，并检查其是否有值。
  - 创建了另一个`Nullable<int>`类型的`number2`，并调用`GetValueOrDefault`方法。

#### TryGetValue 方法的签名

```csharp
public bool TryGetValue(TKey key, out TValue value);
```

**参数**：

 - key：要查找的键，类型为 TKey。
 - value：当方法返回时，如果找到了指定的键，此参数将包含与该键关联的值；如果未找到，此参数将设置为类型 TValue 的默认值。这是一个 out 参数。
 - 返回值：如果字典中包含具有指定键的元素，则返回 true；否则返回 false。
### 3. `DiscountCalculator.cs`
```csharp
using System;

namespace HelloWord
{
    public class DiscountCalculator<T> where T : Product
    {

    }

    // where T : IComparable
    // where T : Product
    // where T : struct
    // where T : new()
    public class Utility<T> where T : IComparable, new()
    {
        public int FindMax(int a, int b)
        {
            return a > b ? a : b;
        }

        public T FindMax(T a, T b)
        {
            return a.CompareTo(b) > 0 ? a : b;
        }

        public void DoSomething()
        {
            var obj = new T();
        }
    }
}
```
- **功能概述**：
  - `DiscountCalculator<T>`是一个泛型类，其中类型参数`T`必须是`Product`类或其子类。
  - `Utility<T>`也是一个泛型类，类型参数`T`必须实现`IComparable`接口，并且必须有一个无参构造函数。该类有两个`FindMax`方法，一个用于比较两个整数，另一个用于比较两个泛型类型的对象。`DoSomething`方法创建了一个`T`类型的对象。

#### 泛型的基本概念
泛型的核心思想是将类型参数化，也就是说在定义类、接口或方法时，不预先指定具体的数据类型，而是使用一个类型参数来代替。在使用这些泛型类型时，再指定具体的类型，这样就可以实现代码的复用。
#### 代码 `public class Utility<T> where T : IComparable, new()` 详细解释

#### 1. `public class Utility<T>`

这部分定义了一个公共的泛型类 `Utility`，其中 `<T>` 表示这是一个泛型类型，`T` 是类型参数，它是一个占位符，代表任意类型。在使用 `Utility` 类时，需要指定具体的类型来替换 `T`。例如，`Utility<int>` 表示使用 `int` 类型作为类型参数。

#### 2. `where T : IComparable, new()`

这是泛型约束，用于限制类型参数 `T` 必须满足的条件。



- `where` 关键字用于引入泛型约束。
- `T : IComparable`：表示类型参数 `T` 必须实现 `IComparable` 接口。`IComparable` 接口定义了一个 `CompareTo` 方法，用于比较对象的大小。通过这个约束，在 `Utility<T>` 类的方法中就可以调用 `T` 类型对象的 `CompareTo` 方法来进行比较操作。例如，`public T FindMax(T a, T b)` 方法中使用了 `a.CompareTo(b)` 来比较两个 `T` 类型对象的大小。
- `T : new()`：表示类型参数 `T` 必须有一个无参数的公共构造函数。这样，在 `Utility<T>` 类的方法中就可以使用 `new T()` 来创建 `T` 类型的对象。例如，`public void DoSomething()` 方法中使用了 `var obj = new T();` 来创建一个 `T` 类型的实例。
### 4. `Book.cs`
```csharp
namespace HelloWord
{
    public class Book : Product
    {
        public string ISBN { get; set; }
    }
}
```
- **功能概述**：定义了一个`Book`类，该类继承自`Product`类，并添加了一个`ISBN`属性。

### 5. `Product.cs`
```csharp
namespace HelloWord
{
    public class Product
    {
        public int Id { get; set; }
        public decimal Price { get; set; }
    }
}
```
- **功能概述**：定义了一个`Product`类，该类有两个属性：`Id`和`Price`。

### 6. `ProductList.cs`
```csharp
using System;

namespace HelloWord
{
    public class ProductList
    {
        public void Add(Product order)
        {
            throw new NotImplementedException();
        }

        public Product this[int index]
        {
            get { throw new NotImplementedException(); }
        }
    }

    public class ObjectList
    {
        public void Add(object o)
        {
            throw new NotImplementedException();
        }

        public object this[int index]
        {
            get { throw new NotImplementedException(); }
        }
    }

    public class GenericList<T>
    {
        public void Add(T o)
        {
            throw new NotImplementedException();
        }

        public T this[int index]
        {
            get { throw new NotImplementedException(); }
        }
    }

    public class Dictionary<TKey, TValue>
    {
        public void Add(TKey key, TValue value)
        {
            throw new NotImplementedException();
        }

        public TValue get(TKey key)
        {
            throw new NotImplementedException(); 
        }
    }
}
```
- **功能概述**：
  - `ProductList`类用于处理`Product`对象的列表，有一个`Add`方法和一个索引器，但具体实现未完成。
  - `ObjectList`类用于处理任意对象的列表，同样有一个`Add`方法和一个索引器，实现未完成。
  - `GenericList<T>`是一个泛型列表类，可用于处理任意类型的对象，有一个`Add`方法和一个索引器，实现未完成。
  - `Dictionary<TKey, TValue>`是一个泛型字典类，用于存储键值对，有一个`Add`方法和一个`get`方法，实现未完成。

### 7. `Nullable.cs`
```csharp
namespace HelloWord
{
    public class Nullable<T> where T : struct
    {
        public Nullable() { }

        private object _value;

        public Nullable(T value)
        {
            _value = value;
        }

        public bool HasValue
        {
            get
            {
                return _value != null;
            }
        }

        public T GetValueOrDefault()
        {
            if (HasValue)
            {
                return (T)_value;
            }
            return default(T);
        }
    }
}
```
- **功能概述**：定义了一个泛型类`Nullable<T>`，其中类型参数`T`必须是值类型（结构体）。该类有一个私有字段`_value`用于存储值，一个无参构造函数和一个带参构造函数。`HasValue`属性用于检查是否有值，`GetValueOrDefault`方法用于获取值，如果没有值则返回默认值。

# nullables

```csharp
using System;

namespace HelloWord
{
    class Program
    {
        static void Main(string[] args)
        {
            //DateTime date = null;
            //Nullable<DateTime> date = null;
            //DateTime? date = null;

            //Console.WriteLine(date.GetValueOrDefault());
            //Console.WriteLine(date.HasValue);
            //Console.WriteLine(date.Value);

            DateTime? date = new DateTime(2099, 1, 1);
            DateTime date2 = date.GetValueOrDefault();
            DateTime? date3 = date2;

            if(date3 != null)
            {
                Console.WriteLine(date3.GetValueOrDefault());
            } else
            {
                Console.WriteLine(DateTime.Today);
            }

            var result = date3 ?? DateTime.Today;
            Console.WriteLine(result);

            Console.Read();
        }
    }
}
```
## 整体功能概述
此代码的主要功能是演示在 C# 里如何运用可空值类型（Nullable Types），并且展示了对可空类型变量的操作，像获取默认值、检查是否有值以及使用空合并运算符（`??`）等。
### 1. 可空类型的声明注释
```csharp
//DateTime date = null;
//Nullable<DateTime> date = null;
//DateTime? date = null;
```
- `DateTime date = null;`：这行代码会引发编译错误，因为 `DateTime` 是值类型，不能直接赋值为 `null`。
- `Nullable<DateTime> date = null;`：使用 `Nullable<T>` 泛型类型声明一个可空的 `DateTime` 变量，能够赋值为 `null`。
- `DateTime? date = null;`：这是 `Nullable<DateTime>` 的简写形式，同样可以声明一个可空的 `DateTime` 变量。

### 2. 对可空类型变量的操作注释
```csharp
//Console.WriteLine(date.GetValueOrDefault());
//Console.WriteLine(date.HasValue);
//Console.WriteLine(date.Value);
```
- `date.GetValueOrDefault()`：返回可空类型变量的值，若该变量为 `null`，则返回该类型的默认值。
- `date.HasValue`：检查可空类型变量是否有值，返回一个布尔值。
- `date.Value`：获取可空类型变量的值，若该变量为 `null`，则会抛出 `InvalidOperationException` 异常。

### 3. 可空类型变量的初始化与操作
```csharp
DateTime? date = new DateTime(2099, 1, 1);
DateTime date2 = date.GetValueOrDefault();
DateTime? date3 = date2;
```
- `DateTime? date = new DateTime(2099, 1, 1);`：声明并初始化一个可空的 `DateTime` 变量 `date`，其值为 2099 年 1 月 1 日。
- `DateTime date2 = date.GetValueOrDefault();`：获取 `date` 的值，若 `date` 为 `null`，则返回 `DateTime` 类型的默认值，然后将结果赋值给非可空的 `DateTime` 变量 `date2`。
- `DateTime? date3 = date2;`：将非可空的 `DateTime` 变量 `date2` 赋值给可空的 `DateTime` 变量 `date3`。

### 4. 条件判断与输出
```csharp
if(date3 != null)
{
    Console.WriteLine(date3.GetValueOrDefault());
} else
{
    Console.WriteLine(DateTime.Today);
}
```
- `if(date3 != null)`：检查 `date3` 是否有值。
- `Console.WriteLine(date3.GetValueOrDefault());`：若 `date3` 有值，则输出该值。
- `Console.WriteLine(DateTime.Today);`：若 `date3` 为 `null`，则输出当前日期。

### 5. 空合并运算符的使用
```csharp
var result = date3 ?? DateTime.Today;
Console.WriteLine(result);
```
- `var result = date3 ?? DateTime.Today;`：使用空合并运算符 `??`，若 `date3` 有值，则将其值赋给 `result`；若 `date3` 为 `null`，则将 `DateTime.Today` 的值赋给 `result`。
- `Console.WriteLine(result);`：输出 `result` 的值。

# extesion

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

namespace HelloWord
{
    class Program
    {
        static void Main(string[] args)
        {
            string hello = "Hello World";
            Console.WriteLine(hello.ShortTerm(2)); // He

            IEnumerable list = new List<int>();
            //list.

            Console.Read();
        }
    }

    public static class StringExtension
    {
        public static string ShortTerm(this string str, int num)
        {
            return str.Substring(0, num);
        }
    }

    //class RichString: String
    //{

    //}

}
```

## 解释
### 1. 命名空间和引用
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

namespace HelloWord
{
    // 类和方法定义
}
```
- **`using` 语句**：这些语句用于引入不同的命名空间，以便在代码中可以直接使用这些命名空间下的类和类型，而无需使用完整的命名空间限定。
  - `System`：包含了许多基础的类型和功能，如 `Console` 类用于控制台输入输出。
  - `System.Collections`：提供了各种集合类和接口，如 `IEnumerable`。
  - `System.Collections.Generic`：提供了泛型集合类，如 `List<T>`。
  - `System.Linq`：提供了用于查询和操作数据的语言集成查询（LINQ）功能。
- **`namespace HelloWord`**：定义了一个名为 `HelloWord` 的命名空间，用于组织代码，避免命名冲突。

### 2. `Program` 类和 `Main` 方法
```csharp
class Program
{
    static void Main(string[] args)
    {
        string hello = "Hello World";
        Console.WriteLine(hello.ShortTerm(2)); // He

        IEnumerable list = new List<int>();
        //list.

        Console.Read();
    }
}
```
- **`Program` 类**：这是一个包含程序入口点的类。
- **`Main` 方法**：这是程序的入口点，程序从这里开始执行。`static` 关键字表示该方法属于类而不是类的实例，`void` 表示该方法不返回任何值，`string[] args` 是命令行参数。
  - `string hello = "Hello World";`：定义了一个字符串变量 `hello` 并初始化为 `"Hello World"`。
  - `Console.WriteLine(hello.ShortTerm(2));`：调用了 `hello` 字符串的 `ShortTerm` 扩展方法，传入参数 `2`，并将结果输出到控制台。`ShortTerm` 方法会截取字符串的前 `2` 个字符，所以输出为 `"He"`。
  - `IEnumerable list = new List<int>();`：创建了一个 `List<int>` 类型的实例，并将其赋值给一个 `IEnumerable` 类型的变量 `list`。`IEnumerable` 是一个接口，它表示一个可枚举的集合。
  - `Console.Read();`：暂停程序的执行，等待用户输入一个字符。

### 3. 字符串扩展方法
```csharp
public static class StringExtension
{
    public static string ShortTerm(this string str, int num)
    {
        return str.Substring(0, num);
    }
}
```
- **`StringExtension` 类**：这是一个静态类，用于定义扩展方法。扩展方法允许你向现有类型添加新的方法，而无需修改该类型的源代码。
- **`ShortTerm` 方法**：这是一个扩展方法，用于截取字符串的前 `num` 个字符。`this string str` 表示该方法是 `string` 类型的扩展方法，`str` 是要操作的字符串，`num` 是要截取的字符数。`str.Substring(0, num)` 调用了 `string` 类的 `Substring` 方法来截取字符串。

# dynamic

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

namespace HelloWord
{
    public class Excel
    {
        public string Table { get; set; }
        public void ShowTable()
        {
            Console.WriteLine("打印excel表格");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            //object obj = new Excel();
            //var obj = new Excel();
            dynamic obj = new Excel();
            obj.GetHashCode();
            obj.ShowTable();

            //var methodInfo = obj.GetType().GetMethod("ShowTable");
            //methodInfo.Invoke(obj, null);

            obj = "hello world";

            Console.Read();
        }
    }
}
```
## 解释
### 1. `Program` 类和 `Main` 方法
```csharp
class Program
{
    static void Main(string[] args)
    {
        //object obj = new Excel();
        //var obj = new Excel();
        dynamic obj = new Excel();
        obj.GetHashCode();
        obj.ShowTable();

        //var methodInfo = obj.GetType().GetMethod("ShowTable");
        //methodInfo.Invoke(obj, null);

        obj = "hello world";

        Console.Read();
    }
}
```
- `class Program`：定义了一个名为 `Program` 的类，通常在 C# 控制台应用程序中，`Program` 类包含程序的入口点。
- `static void Main(string[] args)`：定义了程序的入口点方法 `Main`。`static` 表示该方法属于类本身，而不是类的实例；`void` 表示该方法不返回任何值；`string[] args` 是一个字符串数组，用于接收命令行参数。

#### 变量声明和动态类型
```csharp
dynamic obj = new Excel();
```
- `dynamic` 关键字：用于声明一个动态类型的变量 `obj`。动态类型的变量在编译时不会进行类型检查，而是在运行时确定其类型。这里将 `obj` 初始化为 `Excel` 类的一个实例。

#### 方法调用
```csharp
obj.GetHashCode();
obj.ShowTable();
```
- `obj.GetHashCode()`：调用 `obj` 的 `GetHashCode` 方法。由于 `obj` 是动态类型，编译器不会检查 `GetHashCode` 方法是否存在，而是在运行时确定 `obj` 的实际类型（`Excel` 类），并调用该类型的 `GetHashCode` 方法。
- `obj.ShowTable()`：调用 `obj` 的 `ShowTable` 方法，同样在运行时确定类型并调用该方法，最终在控制台输出 “打印excel表格”。

#### 反射调用方法（注释部分）
```csharp
//var methodInfo = obj.GetType().GetMethod("ShowTable");
//methodInfo.Invoke(obj, null);
```
- 这部分代码使用了反射机制。反射允许在运行时检查和操作类型、方法和属性。
  - `obj.GetType().GetMethod("ShowTable")`：获取 `obj` 类型的 `ShowTable` 方法的 `MethodInfo` 对象。
  - `methodInfo.Invoke(obj, null)`：调用 `ShowTable` 方法，第一个参数是要调用方法的对象，第二个参数是传递给方法的参数数组，这里为 `null` 表示不传递任何参数。

#### 动态类型的重新赋值
```csharp
obj = "hello world";
```
- 将 `obj` 重新赋值为一个字符串 `"hello world"`。由于 `obj` 是动态类型，它可以在运行时改变其类型。

# metadata
## 代码文件分析
### 1. `HelloWord/List.cs`
```csharp
using System;

namespace HelloWord
{
    public class List
    {
        public void Add()
        {
            Console.WriteLine("ddddddd");
        }
    }
}
```
### 2. `HelloWord/Program.cs`
```csharp
using System;
using System.Reflection;

namespace HelloWord
{
    class Program
    {
        static void Main(string[] args)
        {
            // 定位类                      命名空间.类名，   项目名称
            const string classLocation = "HelloWord.List, HelloWord";

            // 获取 List 类型对象
            Type objectType = Type.GetType(classLocation);

            // 通过类型实例化
            object obj = Activator.CreateInstance(objectType);

            // 调用“Add”方法
            MethodInfo add = objectType.GetMethod("Add");
            add.Invoke(obj, null);

            Console.Read();
        }
    }
}
```
- **命名空间引用 (`using`)**：
  - `using System;`：引入 `System` 命名空间，该命名空间包含了许多常用的基础类型和功能，如 `Console` 类。
  - `using System.Reflection;`：引入 `System.Reflection` 命名空间，该命名空间提供了反射相关的类和方法。
- **主类 (`Program`)**：包含一个静态的 `Main` 方法，这是 C# 控制台应用程序的入口点。
- **`Main` 方法内部步骤：**
  1. **定义类的位置 (`classLocation`)**：
     ```csharp
     const string classLocation = "HelloWord.List, HelloWord";
     ```
     这里的字符串 `"HelloWord.List, HelloWord"` 包含了类的完整名称（命名空间.类名）和程序集名称。这是为了让反射机制能够准确地找到所需的类。
  2. **获取类型对象 (`objectType`)**：
     ```csharp
     Type objectType = Type.GetType(classLocation);
     ```
     `Type.GetType` 方法根据提供的类位置字符串获取对应的 `Type` 对象。`Type` 对象包含了类的所有元数据信息。
  3. **实例化对象 (`obj`)**：
     ```csharp
     object obj = Activator.CreateInstance(objectType);
     ```
     `Activator.CreateInstance` 方法根据 `Type` 对象创建类的实例。由于反射是在运行时进行的，所以返回的是 `object` 类型，需要时可以进行强制类型转换。
  4. **获取方法信息 (`add`)**：
     ```csharp
     MethodInfo add = objectType.GetMethod("Add");
     ```
     `GetMethod` 方法从 `Type` 对象中获取名为 `"Add"` 的方法的 `MethodInfo` 对象。`MethodInfo` 对象包含了方法的元数据信息，如方法名、参数类型等。
  5. **调用方法 (`add.Invoke`)**：
     ```csharp
     add.Invoke(obj, null);
     ```
     `Invoke` 方法用于调用 `MethodInfo` 对象所代表的方法。第一个参数是要调用方法的对象实例，第二个参数是传递给方法的参数数组。由于 `Add` 方法没有参数，所以传递 `null`。
  6. **等待用户输入 (`Console.Read`)**：
     ```csharp
     Console.Read();
     ```
     这行代码会暂停程序的执行，直到用户按下任意键，防止控制台窗口在输出结果后立即关闭。

# exception

```csharp
using System;
using System.IO;

namespace HelloWord
{
    public class Calculator
    {
        public int Divde(int numerator, int denomenator)
        {
            return numerator / denomenator;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {

            StreamReader streamReader = null;

            try
            {
                string desktopPath = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
                string fullName = Path.Combine(desktopPath, "123.txt");
                streamReader = new StreamReader(fullName);
                var connect = streamReader.ReadToEnd();

                // 操作文件

                // 人为抛出异常
                throw new Exception("Oops");

                // 关闭文件，回收垃圾
                streamReader.Dispose();
            }
            catch (Exception ex)
            {
                Console.WriteLine("系统异常");
            }
            finally
            {
                if (streamReader != null)
                {
                    streamReader.Dispose();
                    Console.WriteLine("文件回收");
                }
            }

            Console.Read();
        }
    }
}
```
## 代码逐行分析

### 1. 引用命名空间
```csharp
using System;
using System.IO;
```
- `using System;`：引入 `System` 命名空间，该命名空间包含了许多基础的类和类型，如 `Console`、`Exception` 等。
- `using System.IO;`：引入 `System.IO` 命名空间，该命名空间提供了用于处理文件和流的类，如 `StreamReader`。
### 2. 读取文件操作
```csharp
StreamReader streamReader = null;

try
{
    string desktopPath = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
    string fullName = Path.Combine(desktopPath, "123.txt");
    streamReader = new StreamReader(fullName);
    var connect = streamReader.ReadToEnd();

    // 操作文件

    // 人为抛出异常
    throw new Exception("Oops");

    // 关闭文件，回收垃圾
    streamReader.Dispose();
}
```
- `StreamReader streamReader = null;`：声明一个 `StreamReader` 类型的变量 `streamReader` 并初始化为 `null`，用于读取文件。
- `string desktopPath = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);`：获取当前用户桌面的路径。
- `string fullName = Path.Combine(desktopPath, "123.txt");`：将桌面路径和文件名 `123.txt` 组合成完整的文件路径。
- `streamReader = new StreamReader(fullName);`：创建一个 `StreamReader` 对象，用于读取指定路径的文件。
- `var connect = streamReader.ReadToEnd();`：读取文件的全部内容并存储在变量 `connect` 中。不过，这里存在一个拼写错误，变量名 `connect` 可能应该为 `content`。
- `throw new Exception("Oops");`：人为抛出一个异常，异常消息为 "Oops"。
- `streamReader.Dispose();`：释放 `StreamReader` 对象占用的资源，但由于前面抛出了异常，这行代码不会被执行。

### 3. 异常处理
```csharp
catch (Exception ex)
{
    Console.WriteLine("系统异常");
}
```
- `catch (Exception ex)`：捕获所有类型的异常，并将异常对象存储在变量 `ex` 中。
- `Console.WriteLine("系统异常");`：当捕获到异常时，在控制台输出 "系统异常"。

### 4. 资源释放
```csharp
finally
{
    if (streamReader != null)
    {
        streamReader.Dispose();
        Console.WriteLine("文件回收");
    }
}
```
- `finally`：无论是否发生异常，`finally` 块中的代码都会被执行。
- `if (streamReader != null)`：检查 `streamReader` 是否为 `null`，如果不为 `null`，则调用 `Dispose` 方法释放资源，并在控制台输出 "文件回收"。

# reflection

```csharp
using System;
using System.IO;
using System.Runtime.Loader;
using System.Collections;
using System.Collections.Generic;
using Computer.SDK;

namespace Computer
{
    class Program
    {
        static void Main(string[] args)
        {

            // 程序路径
            var USBInterface = Path.Combine(Environment.CurrentDirectory, "USB");
            Console.WriteLine(USBInterface);

            // 文件读写操作
            var dllFiles = Directory.GetFiles(USBInterface);

            // USB 设备列表
            var devicesList = new List<IUSB>();

            foreach (var dll in dllFiles)
            {
                var assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(dll);
                var types = assembly.GetTypes();
                foreach (var type in types)
                {
                    var interfaceList = type.GetInterfaces();
                    foreach (var i in interfaceList)
                    {
                        if (i.Name == "IUSB")
                        {
                            IUSB device = (IUSB)Activator.CreateInstance(type);
                            devicesList.Add(device);
                        }
                    }
                }
            }

            foreach (var devices in devicesList)
            {
                devices.GetInfo();
                devices.Read();
                devices.Wirte();
            }

            Console.Read();
        }
    }
}
```
## 解释
### 1. 确定程序路径
```csharp
// 程序路径
var USBInterface = Path.Combine(Environment.CurrentDirectory, "USB");
Console.WriteLine(USBInterface);
```
- `Path.Combine` 方法用于将当前程序的工作目录（通过 `Environment.CurrentDirectory` 获取）和 `"USB"` 文件夹名称组合成一个完整的路径。这样做的目的是找到存储 USB 设备相关 DLL 文件的目录。
- `Console.WriteLine(USBInterface)` 用于在控制台输出组合后的路径，方便调试时确认路径是否正确。

### 2. 获取 DLL 文件列表
```csharp
// 文件读写操作
var dllFiles = Directory.GetFiles(USBInterface);
```
- `Directory.GetFiles` 方法会返回指定目录（即 `USBInterface` 所代表的目录）下的所有文件的完整路径，并将这些路径存储在 `dllFiles` 数组中。这里没有指定文件扩展名筛选条件，所以会获取该目录下的所有文件。

### 3. 初始化 USB 设备列表
```csharp
// USB 设备列表
var devicesList = new List<IUSB>();
```
- 创建一个 `List<IUSB>` 类型的列表 `devicesList`，用于存储所有实现了 `IUSB` 接口的类的实例。

### 4. 遍历 DLL 文件并加载程序集
```csharp
foreach (var dll in dllFiles)
{
    var assembly = AssemblyLoadContext.Default.LoadFromAssemblyPath(dll);
    var types = assembly.GetTypes();
    // ...
}
```
- 使用 `foreach` 循环遍历 `dllFiles` 数组中的每个 DLL 文件路径。
- `AssemblyLoadContext.Default.LoadFromAssemblyPath(dll)` 方法用于动态加载指定路径的 DLL 文件，并将其作为一个程序集（`Assembly` 对象）返回。
- `assembly.GetTypes()` 方法会返回该程序集中定义的所有类型（类、接口等），并将这些类型存储在 `types` 数组中。

### 5. 筛选实现 `IUSB` 接口的类型并创建实例
```csharp
foreach (var type in types)
{
    var interfaceList = type.GetInterfaces();
    foreach (var i in interfaceList)
    {
        if (i.Name == "IUSB")
        {
            IUSB device = (IUSB)Activator.CreateInstance(type);
            devicesList.Add(device);
        }
    }
}
```
- 对于程序集中的每个类型，使用 `type.GetInterfaces()` 方法获取该类型实现的所有接口，并将这些接口存储在 `interfaceList` 数组中。
- 遍历 `interfaceList` 数组，检查每个接口的名称是否为 `"IUSB"`。
- 如果找到实现了 `IUSB` 接口的类型，使用 `Activator.CreateInstance(type)` 方法创建该类型的实例，并将其转换为 `IUSB` 接口类型，存储在 `device` 变量中。
- 最后，将创建的实例添加到 `devicesList` 列表中。


