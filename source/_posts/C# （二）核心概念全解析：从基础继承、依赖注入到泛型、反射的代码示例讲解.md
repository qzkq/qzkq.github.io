
---
title: C# （二）核心概念全解析：从基础继承、依赖注入到泛型、反射的代码示例讲解
date: 2026-05-24 08:00:00
tags:
  - "C#"
  - ".NET"
  - "依赖注入"
  - "泛型"
  - "反射"
categories:
  - "编程语言"
---

# delegate 委托

在C#中，`delegate`（委托）是一种引用类型，它可以封装一个或多个方法，并且可以像调用普通方法一样调用这些封装的方法。委托类似于函数指针，但它是类型安全的，并且支持多播（可以包含多个方法）。下面结合你提供的代码详细解释委托的相关概念。
## 解释
### 1. 委托的定义
在`PhotoProcessor`类中，定义了一个委托`PhotoFilterHandler`：
```csharp
public delegate void PhotoFilterHandler(Photo photo);
```
- **语法**：`delegate`关键字用于声明委托类型。这里定义了一个名为`PhotoFilterHandler`的委托，它返回`void`类型，并且接受一个`Photo`类型的参数。
- **作用**：委托定义了一种方法签名，任何符合该签名的方法都可以被封装到这个委托类型的实例中。

### 2. 委托的使用
在`Program`类的`Main`方法中，使用了`PhotoFilterHandler`委托：
```csharp
PhotoFilterHandler filterHandler = filters.ApplyBrightness;
filterHandler += filters.ApplyContrast;
filterHandler += filters.Resize;
filterHandler += RedEyesRemovalFilter;
```
- **创建委托实例**：首先，将`filters.ApplyBrightness`方法赋值给`filterHandler`委托实例。这表明`filterHandler`现在引用了`ApplyBrightness`方法。
- **多播委托**：使用`+=`运算符可以将多个方法添加到委托实例中，形成一个委托链。这里依次将`filters.ApplyContrast`、`filters.Resize`和`RedEyesRemovalFilter`方法添加到`filterHandler`委托实例中。当调用这个委托实例时，会依次调用委托链中的所有方法。

### 3. 委托的调用
在`PhotoProcessor`类的`Process`方法中，调用了委托实例：
```csharp
public void Process(Photo photo, PhotoFilterHandler filterHandler)
{
    filterHandler(photo);
    photo.Save();
}
```
- **语法**：直接使用委托实例的名称，并传递相应的参数来调用委托。这里调用`filterHandler(photo)`会依次调用委托链中的所有方法，并将`photo`对象作为参数传递给每个方法。

### 4. 委托的优势
- **解耦**：委托允许将方法作为参数传递，从而实现了代码的解耦。在`Process`方法中，它只关心委托的签名，而不关心具体的方法实现。这使得代码更加灵活，可以在运行时动态地选择要执行的方法。
- **多播**：委托支持多播，可以将多个方法组合在一起执行。这在需要同时执行多个操作的场景中非常有用，例如在照片处理中，可以同时应用多个滤镜。

```csharp
namespace HelloWorld
{
    public class PhotoProcessor
    {
        public delegate void PhotoFilterHandler(Photo photo);

        public void Process(Photo photo, PhotoFilterHandler filterHandler)
        {
            filterHandler(photo);
            photo.Save();
        }
    }
}

# Action<Photo> 是 .NET 框架预定义的泛型委托类型。它是在 System 命名空间中定义的，可用于封装一个接受单个 Photo 类型参数且不返回值的方法。
using System;

namespace HelloWorld
{
    public class PhotoProcessor
    {
        public void Process(Photo photo, Action<Photo> filterHandler)
        {
            filterHandler(photo);
            photo.Save();
        }
    }
}
```
# Timer

```csharp
using System;
using System.Timers;

namespace HelloWorld
{
    public class Alex
    {
        internal void AlarmEventHandler(object sender, ElapsedEventArgs e)
        {
            Console.WriteLine("闹钟响了, 我不管");
        }
    }

    public class RoomMate
    {
        public int RageValue { get; set; }

        internal void AlarmEventHandler(object sender, ElapsedEventArgs e)
        {
            RageValue += 25;
            if(RageValue >= 100)
            {
                Console.WriteLine("受不了了");
                ((Timer)sender).Stop();
            }
            Console.WriteLine("闹钟响了, 我也不管");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Timer timer = new Timer();
            timer.Interval = 1000;

            var alex = new Alex();
            var roomMate = new RoomMate();
            timer.Elapsed += alex.AlarmEventHandler;
            timer.Elapsed += roomMate.AlarmEventHandler;

            timer.Start();

            Console.Read();
        }
    }
}
```

## 解释

### 1. 命名空间引用
```csharp
using System;
using System.Timers;
```
- `using System;`：引入 `System` 命名空间，它是 .NET 框架中最基础的命名空间，包含了许多常用的类型和功能，例如 `Console` 类用于控制台输入输出操作。
- `using System.Timers;`：引入 `System.Timers` 命名空间，该命名空间提供了 `Timer` 类，用于创建定时器。
### 2. `Alex` 类
```csharp
public class Alex
{
    internal void AlarmEventHandler(object sender, ElapsedEventArgs e)
    {
        Console.WriteLine("闹钟响了, 我不管");
    }
}
```
- `public class Alex`：定义了一个公共类 `Alex`，意味着这个类可以被其他命名空间中的代码访问。
- `internal void AlarmEventHandler(object sender, ElapsedEventArgs e)`：
  - `internal` 访问修饰符表示该方法只能在当前程序集（通常是同一个项目）内被访问。
  - `AlarmEventHandler` 是一个事件处理方法，用于处理定时器的 `Elapsed` 事件。
  - `object sender` 参数表示触发事件的对象，在这个场景中就是定时器实例。
  - `ElapsedEventArgs e` 参数包含了事件相关的信息，例如事件触发的时间。
  - `Console.WriteLine("闹钟响了, 我不管");`：当定时器触发 `Elapsed` 事件时，在控制台输出 “闹钟响了, 我不管”。

### 3. `RoomMate` 类
```csharp
public class RoomMate
{
    public int RageValue { get; set; }

    internal void AlarmEventHandler(object sender, ElapsedEventArgs e)
    {
        RageValue += 25;
        if(RageValue >= 100)
        {
            Console.WriteLine("受不了了");
            ((Timer)sender).Stop();
        }
        Console.WriteLine("闹钟响了, 我也不管");
    }
}
```
- `public class RoomMate`：定义了一个公共类 `RoomMate`。
- `public int RageValue { get; set; }`：定义了一个公共的整数属性 `RageValue`，用于存储室友的愤怒值。`{ get; set; }` 是自动属性的语法，编译器会自动生成对应的 getter 和 setter 方法。
- `internal void AlarmEventHandler(object sender, ElapsedEventArgs e)`：
  - 同样是处理定时器 `Elapsed` 事件的方法。
  - `RageValue += 25;`：每次定时器触发事件时，将室友的愤怒值增加 25。
  - `if(RageValue >= 100)`：检查室友的愤怒值是否达到或超过 100。
    - `Console.WriteLine("受不了了");`：如果愤怒值达到或超过 100，在控制台输出 “受不了了”。
    - `((Timer)sender).Stop();`：将 `sender` 对象强制转换为 `Timer` 类型，然后调用 `Stop()` 方法停止定时器。
  - `Console.WriteLine("闹钟响了, 我也不管");`：无论愤怒值是否达到阈值，每次定时器触发时都会在控制台输出 “闹钟响了, 我也不管”。

### 4. `Program` 类和 `Main` 方法
```csharp
class Program
{
    static void Main(string[] args)
    {
        Timer timer = new Timer();
        timer.Interval = 1000;

        var alex = new Alex();
        var roomMate = new RoomMate();
        timer.Elapsed += alex.AlarmEventHandler;
        timer.Elapsed += roomMate.AlarmEventHandler;

        timer.Start();

        Console.Read();
    }
}
```
- `class Program`：定义了一个类 `Program`，通常在 C# 控制台应用程序中，`Program` 类包含程序的入口点。
- `static void Main(string[] args)`：
  - `static` 关键字表示该方法属于类本身，而不是类的实例。
  - `void` 表示该方法不返回任何值。
  - `Main` 方法是 C# 控制台应用程序的入口点，程序从这里开始执行。
  - `args` 是一个字符串数组，用于接收命令行参数。
- `Timer timer = new Timer();`：创建一个新的 `Timer` 对象。
- `timer.Interval = 1000;`：设置定时器的触发间隔为 1000 毫秒（即 1 秒），意味着定时器每秒会触发一次 `Elapsed` 事件。
- `var alex = new Alex();`：创建 `Alex` 类的一个实例。
- `var roomMate = new RoomMate();`：创建 `RoomMate` 类的一个实例。
- `timer.Elapsed += alex.AlarmEventHandler;`：将 `Alex` 类的 `AlarmEventHandler` 方法注册到定时器的 `Elapsed` 事件上，当定时器触发 `Elapsed` 事件时，会调用该方法。
- `timer.Elapsed += roomMate.AlarmEventHandler;`：将 `RoomMate` 类的 `AlarmEventHandler` 方法注册到定时器的 `Elapsed` 事件上，当定时器触发 `Elapsed` 事件时，也会调用该方法。
- `timer.Start();`：启动定时器，开始按照设定的间隔触发 `Elapsed` 事件。
- `Console.Read();`：使程序暂停，等待用户输入一个字符，防止程序在定时器事件处理过程中立即退出。


