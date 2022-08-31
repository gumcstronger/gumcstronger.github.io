---
layout:     post
title:      "C#服务器编程(未完待续)"
subtitle:   " \"C#服务器编程：异步编程、Acotr模型、Context­\""
date:       2022-08-09 10:15:00
author:     "Gumc"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 游戏开发
    - Server
--- 

- [C#语言相关](#c语言相关)
    - [静态构造函数](#静态构造函数)
- [NuGet相关库](#nuget相关库)
    - [ConsoleAppFramework](#consoleappframework)
- [Context](#context)
    - [Wikipedia的解释](#wikipedia的解释)
    - [现在的理解](#现在的理解)
    - [SynchronizationContext(同步上下文)做什么？](#synchronizationcontext同步上下文做什么)
    - [以Geek的RuntimeContext来看](#以geek的runtimecontext来看)
- [异步编程](#异步编程)
    - [什么是异步](#什么是异步)
    - [基于任务的异步模式（TAP）](#基于任务的异步模式tap)
    - [Task](#task)
      - [Task的相关内容](#task的相关内容)
    - [async/await关键字](#asyncawait关键字)
      - [await方法做了什么](#await方法做了什么)
      - [什么是awaiter?](#什么是awaiter)
      - [await和Task.Wait一样吗？不一样](#await和taskwait一样吗不一样)
      - [task.Result和task.GetAwaiter().GetResult()有区别吗？](#taskresult和taskgetawaitergetresult有区别吗)
      - [“await”是如何关联到当前SynchronizationContext](#await是如何关联到当前synchronizationcontext)
    - [ET中关于异步编程的介绍](#et中关于异步编程的介绍)
    - [ETTASK](#ettask)
    - [Geek](#geek)
- [Actor](#actor)
    - [网上关于Actor的解释](#网上关于actor的解释)
    - [所以Actor到底做什么？](#所以actor到底做什么)
    - [C#怎么用TPL DataFlow构建Actor？](#c怎么用tpl-dataflow构建actor)
    - [Actor的实现是做了什么？](#actor的实现是做了什么)
- [序列化工具](#序列化工具)
    - [MessagePack使用](#messagepack使用)
    - [protobuf-net使用](#protobuf-net使用)
- [Net](#net)
    - [Microsoft.AspNetCore.Builder.WebApplication](#microsoftaspnetcorebuilderwebapplication)
    - [Microsoft.AspNetCore.Mvc.Cors](#microsoftaspnetcoremvccors)
    - [System.Net.Http.HttpClient](#systemnethttphttpclient)

# C#语言相关

### 静态构造函数

> 不能从程序中显示调用静态构造函数，系统会自动调用它们，在类的任何实例被创建之前、类的任何静态成员被引用之前，例如new个对象的时候，系统会先调用到静态构造函数(在已经定义的情况下)，然后在调用默认构造函数


# NuGet相关库

### ConsoleAppFramework
>ConsoleAppFramework用于创建CLI（命令行界面）工具、守护进程和多批处理应用程序

```C#
    /* 直接使用 */
    ConsoleApp.Run(args, (string name) => Console.WriteLine($"Hello {name}"));

    /* 注册命令 */
    // AddCommands register as command.
    // echo --msg --repeat(default = 3)
    // sum [x] [y]
    var app = ConsoleApp.Create(args);
    app.AddCommands<Foo>();
    app.Run();

    public class Foo : ConsoleAppBase
    {
        public void Echo(string msg, int repeat = 3)
        {
            for (var i = 0; i < repeat; i++)
            {
                Console.WriteLine(msg);
            }
        }

        public void Sum([Option(0)]int x, [Option(1)]int y)
        {
            Console.WriteLine((x + y).ToString());
        }
    }
```

# Context

> 以前在Nodejs开发时经常看到整个项目会包含一个唯一的“Context”上下文,一直直观的理解是当前项目(进程/线程/程序)运行时所有的数据or对象。  
> 记得在Java开发时就经常出现Context，似乎万物皆是Context的样子，真是奇怪。

### Wikipedia的解释 

>In computer science, a task context (process, thread ...) is the minimal set of data used by this task that must be saved to allow a task interruption at a given date and a continuation of this task at the point it has been interrupted and at an arbitrary future date. The concept of context assumes significance in the case of interruptible tasks, wherein upon being interrupted, the processor saves the context and proceeds to serve the Interrupt service routine. Thus the smaller the context, the smaller is the latency. These data are located in:

>Processor registers
Memory used by the task
On some operating systems, control registers used by the system to manage the task
The storage memory (files) is not concerned by the "task context" in the case of a context switch, even if this can be stored for some uses (checkpointing).

>在计算机科学中，任务上下文（进程、线程……）是该任务使用的最小数据集，必须保存这些数据以允许在给定日期中断任务并在该点继续该任务中断并在任意未来日期。上下文的概念在可中断任务的情况下具有重要意义，其中在被中断时，处理器保存上下文并继续为中断服务程序提供服务。因此，上下文越小，延迟就越小。这些数据位于：
处理器寄存器
任务使用的内存
在某些操作系统上，系统使用控制寄存器来管理任务。
在上下文切换的情况下，存储内存（文件）与“任务上下文”无关，即使它可以存储用于某些用途（检查点）


### 现在的理解
>Context是涉及中断情况下如进程线程时保存的数据对象。  
 所以叫Context时应该是进程、线程等涉及中断异步类才能称呼使用或者称呼为上下文。


### [SynchronizationContext(同步上下文)](https://devblogs.microsoft.com/pfxteam/executioncontext-vs-synchronizationcontext/)做什么？
>SynchronizationContext 只是一种抽象，它表示您想要在其中执行某些工作的特定环境。
>SynchronizationContext提供了一个虚Post方法，该方法只接收一个委托，并在任何地点，任何时间运行它，当然SynchronizationContext的实现要认为是合适的。WinForm提供了WindowsFormSynchronizationContext类，该类重写了Post方法来调用Control.BeginInvoke。WPF提供了DispatcherSynchronizationContext类，该类重写Post方法来调用Dispatcher.BeginInvoke，等等

```C#
    /* 专门针对WinForm编写组件 */
    public static void DoWork(SynchronizationContext sc)
    {
        ThreadPool.QueueUserWorkItem(delegate
        {
            … // 在线程池中执行
            
            sc.Post(delegate
            {
                … // 在UI线程中执行
            }, null);
        });
    }
```
>我的理解ET用SynchronizationContext是为何线程安全，保证拿到正确的结果。

### 以Geek的RuntimeContext来看

RuntimeContext是使用AsyncLocal, 而AsyncLocal实际上是使用了ExecutionContext来作为线程本地存储


# 异步编程
> &ensp;以前用Nodejs,可能因为Node天然属性,只需要在意async和await传递,并不需要太关注异步编程。
> &ensp;最近看C#服务器编程，基本都会出现Task、UniTask，ET还专门为此搞了个ETTask。  
> &ensp;问题1：ET/Unity不是单线程的吗,为何还需要搞ETTask？答案是Task不是单线程,ETTask是单线程Task。  
> &ensp;问题2：异步编程必须是多线程吗？不是的，我们更想让异步编程是单线程的。

### 什么是异步
>&ensp;在异步程序中，程序代码不需要按照编写的顺序严格执行。有时需要在一个新的线程中运行一部分代码，有时无需创建新的线程，但为了更好地利用单个线程的能力，需要改变代码的执行顺序。  
>&ensp;当一个方法被调用时，调用者需要等待该方法执行完毕并返回才能继续执行，我们称这个方法是同步方法；当一个方法被调用时立即返回，并获取一个线程执行该方法内部的业务，调用者不用等待该方法执行完毕，我们称这个方法为异步方法。  
>&ensp;异步的好处在于非阻塞(调用线程不会暂停执行去等待子线程完成)，因此我们把一些不需要立即使用结果、较耗时的任务设为异步执行，可以提高程序的运行效率。net4.0在ThreadPool的基础上推出了Task类，微软极力推荐使用Task来执行异步任务，现在C#类库中的异步方法基本都用到了Task；net5.0推出了async/await，让异步编程更为方便

### [基于任务的异步模式（TAP）](https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)

- 区别于.NET1.0异步编程模型（APM 或 IAsyncResult）模式

```C# 
    /* .NET 1.0时期就提出的一种异步模式，并且基于IAsyncResult接口实现BeginXXX和EndXXX类似的方法。
    .net中有很多类实现了该模式(比如HttpWebRequest)，同时我们也可以自定义类来实现APM模式(继承IAsyncResult接口并且实现BeginXXX和EndXXX方法) */
    /// <summary>
    /// 异步
    /// </summary>
    /// <param name="sender"></param>
    /// <param name="e"></param>
    private void button2_Click(object sender, EventArgs e)
    {
       //先清空上一次查询结果
       this.richTextBox1.Text = "";

       var url = this.textBox1.Text.Trim();
       var request = HttpWebRequest.Create(url);
       request.BeginGetResponse(AsyncCallbackImpl, request);//BeginGetResponse,发起异步请求
    }

    /// <summary>
    /// 回调
    /// </summary>
    /// <param name="ar"></param>
    public void AsyncCallbackImpl(IAsyncResult ar)
    {
       HttpWebRequest request = ar.AsyncState as HttpWebRequest;
       var response = request.EndGetResponse(ar);//EndGetResponse,异步请求完成
       var stream = response.GetResponseStream();
       StringBuilder sb = new StringBuilder();
       sb.AppendLine("当前线程Id:" + Thread.CurrentThread.ManagedThreadId);
       using (StreamReader reader = new StreamReader(stream))
       {
           var content = reader.ReadLine();
           sb.AppendLine(content);
           this.richTextBox1.Text = sb.ToString();
       }
    }
```
- 区别于.NET2.0基于事件的异步模式 (EAP) 

```C#
    /* 基于事件的异步模式是.NET 2.0提出的，实现了基于事件的异步模式的类将具有一个或者多个以Async为后缀的方法和对应的Completed事件，并且这些类都支持异步方法的取消、进度报告和报告结果。然而.net中并不是所有的类都支持EAP。
    当调用基于事件的EAP模式的类的XXXAsync方法时，就开始了一个异步操作，并且基于事件的EAP模式是基于APM模式之上的，而APM又是建立在委托之上的。下面的Demo就以BackgroundWorker类来演示如何使用EAP异步。
    */
    BackgroundWorker worker = new BackgroundWorker();
    worker.DoWork += Worker_DoWork;
    worker.RunWorkerCompleted += Worker_RunWorkerCompleted;
    worker.RunWorkerAsync(null);

    private static void Worker_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
    {
        Console.WriteLine("dowork completed");
    }

    private static void Worker_DoWork(object sender, DoWorkEventArgs e)
    {
       Console.WriteLine("dowork");
    }
```

- TAP模式  
    核心是Task和Task<T>，关键字是async和await。  


### Task
  
>Task是在ThreadPool的基础上推出的，我们简单了解下ThreadPool。ThreadPool中有若干数量的线程，如果有任务需要处理时，会从线程池中获取一个空闲的线程来执行任务，任务执行完毕后线程不会销毁，而是被线程池回收以供后续任务使用。当线程池中所有的线程都在忙碌时，又有新任务要处理时，线程池才会新建一个线程来处理该任务，如果线程数量达到设置的最大值，任务会排队，等待其他任务释放线程后再执行。线程池能减少线程的创建，节省开销。
```C#
    /* ThreadPool相对于Thread来说可以减少线程的创建，有效减小系统开销；但是ThreadPool不能控制线程的执行顺序，我们也不能获取线程池内线程取消/异常/完成的通知，即我们不能有效监控和控制线程池中的线程 */
    static void Main(string[] args)
    {
        for (int i = 1; i <=10; i++)
        {
            //ThreadPool执行任务
            ThreadPool.QueueUserWorkItem(new WaitCallback((obj) => {
                Console.WriteLine($"第{obj}个执行任务");
            }),i);
        }
        Console.ReadKey();
    } 
    //结果不是按1，2，3，4，5，6，7，8，9，10来的
```
> ThreadPool的弊端：我们不能控制线程池中线程的执行顺序，也不能获取线程池内线程取消/异常/完成的通知。net4.0在ThreadPool的基础上推出了Task，Task拥有线程池的优点，同时也解决了使用线程池不易控制的弊端。

#### Task的相关内容

- Task的创建：Task.Run/new Task/Task.Factory.StartNew

```C#
    // Task默认使用线程池，也就是后台线程： 当主线程结束时，你创建所有的tasks都会结束。Task对象的核心在于可以用它来监视其过程。
    static void Main(string[] args)
    {
        //1.new方式实例化一个Task，需要通过Start方法启动
        /*可以通过task的构造函数创建“冷”任务（cold task），但开发中很少这么干 */
        Task task = new Task(() =>
        {
            Thread.Sleep(100);
            Console.WriteLine($"hello, task1的线程ID为{Thread.CurrentThread.ManagedThreadId}");
        });
        task.Start();

        //2.Task.Factory.StartNew(Action action)创建和启动一个Task
        Task task2 = Task.Factory.StartNew(() =>
        {
            Thread.Sleep(100);
            Console.WriteLine($"hello, task2的线程ID为{ Thread.CurrentThread.ManagedThreadId}");
        });

        //3.Task.Run(Action action)将任务放在线程池队列，返回并启动一个Task
        /*  在Task.Run之后，我们没有调用Start，因为该方法创建的是“热”任务（hot task） */
        Task task3 = Task.Run(() =>
        {
            Thread.Sleep(100);
            Console.WriteLine($"hello, task3的线程ID为{ Thread.CurrentThread.ManagedThreadId}");
        });
        /* 执行主线程会比其他先打印出来 */
        Console.WriteLine("执行主线程！");
        Console.ReadKey();
        /* 结果：
        执行主线程！
        hello,task3的线程ID为6
        hello,task2的线程ID为5
        hello,task1的线程ID为4 */
    }
```
```C#
    /* 创建有返回值的Task<TResult> */
    static void Main(string[] args)
    {
        ////1.new方式实例化一个Task，需要通过Start方法启动
        Task<string> task = new Task<string>(() =>
        {
            return $"hello, task1的ID为{Thread.CurrentThread.ManagedThreadId}";
        });
        task.Start();

        ////2.Task.Factory.StartNew(Func func)创建和启动一个Task
       Task<string> task2 =Task.Factory.StartNew<string>(() =>
        {
            return $"hello, task2的ID为{ Thread.CurrentThread.ManagedThreadId}";
        });

        ////3.Task.Run(Func func)将任务放在线程池队列，返回并启动一个Task
       Task<string> task3= Task.Run<string>(() =>
        {
            return $"hello, task3的ID为{ Thread.CurrentThread.ManagedThreadId}";
        });
        /* 注意task.Resut获取结果时会阻塞线程，即如果task没有执行完成，会等待task执行完成获取到Result，然后再执行后边的代码 */
        Console.WriteLine("执行主线程！");
        Console.WriteLine(task.Result);
        Console.WriteLine(task2.Result);
        Console.WriteLine(task3.Result);
        Console.ReadKey();
        /* 结果：
        执行主线程！
        hello,task1的线程ID为4
        hello,task2的线程ID为5
        hello,task3的线程ID为6 */
    }
```

- Task的阻塞方法 Task.Wait/Task.WaitAll/Task.WaitAny  
>Thread的Join方法可以阻塞调用线程，但是有一些弊端：①如果我们要实现很多线程的阻塞时，每个线程都要调用一次Join方法；②如果我们想让所有的线程执行完毕(或者任一线程执行完毕)时，立即解除阻塞，使用Join方法不容易实现。Task提供了  Wait/WaitAny/WaitAll  方法，可以更方便地控制线程阻塞。  
task.Wait()  表示等待task执行完毕，功能类似于thead.Join()；  Task.WaitAll(Task[] tasks)  表示只有所有的task都执行完成了再解除阻塞；  Task.WaitAny(Task[] tasks) 表示只要有一个task执行完毕就解除阻塞  

```C#
    static void Main(string[] args)
    {
        Task task1 = new Task(() => {
            Thread.Sleep(500);
            Console.WriteLine("线程1执行完毕！");
        });
        task1.Start();
        Task task2 = new Task(() => {
            Thread.Sleep(1000);
            Console.WriteLine("线程2执行完毕！");
        });
        task2.Start();
        //阻塞主线程。task1,task2都执行完毕再执行主线程
    //执行【task1.Wait();task2.Wait();】可以实现相同功能
        Task.WaitAll(new Task[]{ task1,task2});
        Console.WriteLine("主线程执行完毕！");
        Console.ReadKey();

        /* 执行结果：
        线程1执行完毕
        线程2执行完毕
        主线程执行完毕！ */
    }
```

- Task的延续操作 Task.WhenAny/Task.WhenAll/Task.ContinueWith
> 上边的Wait/WaitAny/WaitAll方法返回值为void，这些方法单纯的实现阻塞线程。我们现在想让所有task执行完毕(或者任一task执行完毕)后，开始执行后续操作，怎么实现呢？这时就可以用到WhenAny/WhenAll方法了，这些方法执行完成返回一个task实例。  task.WhenAll(Task[] tasks)  表示所有的task都执行完毕后再去执行后续的操作， task.WhenAny(Task[] tasks)  表示任一task执行完毕后就开始执行后续操作

```C#
    static void Main(string[] args)
    {
        Task task1 = new Task(() => {
            Thread.Sleep(500);
            Console.WriteLine("线程1执行完毕！");
        });
        task1.Start();
        Task task2 = new Task(() => {
            Thread.Sleep(1000);
            Console.WriteLine("线程2执行完毕！");
        });
        task2.Start();
        //task1，task2执行完了后执行后续操作
        Task.WhenAll(task1, task2).ContinueWith((t) => {
            Thread.Sleep(100);
            Console.WriteLine("执行后续操作完毕！");
        });

        Console.WriteLine("主线程执行完毕！");
        Console.ReadKey();
    }
    /* 执行结果：
    线程1执行完毕
    线程2执行完毕
    主线程执行完毕！
    执行后续操作 */
```
>上边的例子也可以通过 Task.Factory.ContinueWhenAll(Task[] tasks, Action continuationAction)  和 Task.Factory.ContinueWhenAny(Task[] tasks, Action continuationAction) 来实现
```C#
    static void Main(string[] args)
    {
        Task task1 = new Task(() => {
            Thread.Sleep(500);
            Console.WriteLine("线程1执行完毕！");
        });
        task1.Start();
        Task task2 = new Task(() => {
            Thread.Sleep(1000);
            Console.WriteLine("线程2执行完毕！");
        });
        task2.Start();
        //通过TaskFactroy实现
        Task.Factory.ContinueWhenAll(new Task[] { task1, task2 }, (t) =>
        {
            Thread.Sleep(100);
            Console.WriteLine("执行后续操作");
        });

        Console.WriteLine("主线程执行完毕！");
        Console.ReadKey();
    }
    /* 执行结果：
    线程1执行完毕
    线程2执行完毕
    主线程执行完毕！
    执行后续操作 */
```

- Task的任务取消: CancellationTokenSource
>在Task前我们执行任务采用的是Thread,Thread怎么取消任务呢？一般流程是：设置一个变量来控制任务是否停止，如设置一个变量isStop，然后线程轮询查看isStop，如果isStop为true就停止  
Task中有一个专门的类 CancellationTokenSource  来取消任务执行

```C#
    static void Main(string[] args)
    {
        CancellationTokenSource source = new CancellationTokenSource();
        int index = 0;
        //开启一个task执行任务
        Task task1 = new Task(() =>
          {
              while (!source.IsCancellationRequested)
              {
                  Thread.Sleep(1000);
                  Console.WriteLine($"第{++index}次执行，线程运行中...");
              }
          });
        task1.Start();
        //五秒后取消任务执行
        Thread.Sleep(5000);
        //source.Cancel()方法请求取消任务，IsCancellationRequested会变成true
        source.Cancel();
        Console.ReadKey();
    }

    /* 我们可以使用  source.CancelAfter(5000)  实现5秒后自动取消任务，也可以通过  source.Token.Register(Action action)  注册取消任务触发的回调函数，即任务被取消时注册的action会被执行 */
    static void Main(string[] args)
    {
        CancellationTokenSource source = new CancellationTokenSource();
        //注册任务取消的事件
        source.Token.Register(() =>
        {
            Console.WriteLine("任务被取消后执行xx操作！");
        });

        int index = 0;
        //开启一个task执行任务
        Task task1 = new Task(() =>
          {
              while (!source.IsCancellationRequested)
              {
                  Thread.Sleep(1000);
                  Console.WriteLine($"第{++index}次执行，线程运行中...");
              }
          });
        task1.Start();
        //延时取消，效果等同于Thread.Sleep(5000);source.Cancel();
        source.CancelAfter(5000);
        Console.ReadKey();
    }
    /* 执行结果：
    第1次执行，线程运行中...
    第2次执行，线程运行中...
    第3次执行，线程运行中...
    第4次执行，线程运行中...
    任务被取消后执行xx操作！
    第5次执行，线程运行中... */

    /* 第5次执行在取消回调后打印，这是因为，执行取消的时候第5次任务已经通过了while()判断，任务已经执行中了 */
```

- Task的异常

```C#
    /* 与Thread不一样，Task可以很方便的传播异常 如果你的task里面抛出了一个未处理的异常，那么该异常就会重新被抛出给：
    调用了wait()的地方
    访问了Task的Reuslt属性的地方。 */
    Task mytask = Task.Run(()=> { throw null; });
    try{
        mytask.Wait();
    }
    catch (AggregateException aex){
        if (aex.InnerExceptions is NullReferenceException){
            Console.WriteLine("null");
        }
        else{
            throw;
        }
    }
    /* 如果我们不想抛出异常就想知道task有没有发生故障，无需重新抛出异常，通过Task的IsFaulted和IsCanceled属性也可以检测出Task是否发生了故障:
    如果两个属性都返回false,那么没有错误发生。
    如果IsCanceled为true，那就说明一个OperationCanceledException为该Task抛出了。
    如果IsFaulted为true，那么就说明另一个类型的异常被抛出了，而Exception属性也将指明错误。
     */
```

- Coninuation=>awaiter.OnComplete();

```C#
    /* 在Task上调用GetAwaiter会返回一个awaiter对象。它的OnCompleted方法会告诉之前的task：“当结束/发生故障的时候要执行委托” 。
    可以将Continuation附加到已经结束的task上面，此时continuation将会被安排立即执行
    */
    Task<int> mytask = Task.Run(() => 
    { 
        Console.WriteLine("do it");
        return 666;
    });
    var awaiter =  mytask.GetAwaiter();
    awaiter.OnCompleted(()=> 
    {
        int result = awaiter.GetResult();
        Console.WriteLine(result);
    });
    /* 任何可以暴露下列两个方法和一个属性的对象就是awaiter：
    OnCompleted
    GetResult
    一个叫做IsCompleted的bool属性 */

    /* 如果之前的任务发生故障，那么当continuation代码调用awaiter.GetResult()的时候，异常会被重新抛出。
    无需调用GetResult，我们可以直接访问task的Result属性。
    但调用GetResult的好处是，如果task发生故障，那么异常会被直接的抛出，而不是包裹在AggregateException里面，这样的话catch快就简洁了很多。 */
```

- ContinueWith

```C#
    /* Continuewith本身返回一个task，它可以用它来附加更多的Continuation。
    但是，必须直接处理AggregateException：
    如果task发生故障，需要额外的代码来吧Continuation封装（marshal）到UI应用上。
    在非UI上下文中，弱项让Continuation和task执行在同一个线程上，必须制定TaskContinuationOptions.ExecuteSynchronously,否则将它弹回到线程池。
    */
    Task<int> mytask = Task.Run(() =>  
    {    
    Console.WriteLine("do it"); 
        return 666; 
    });  
    mytask.ContinueWith(task=>  { 
        int result = task.Result; 
        Console.WriteLine(result); 
    });
```

- TaskCompletionSource

```C#
    /* TaskCompletionSource也可以用来创建Task
    TaskCompletionSource让你在稍后开始和结束的任意操作中创建Task
    它会为你提供一个可手动执行的“从属”Task
    只是操作合适结束或发生故障

    它对IO-Bound类工作比较理想
    可以获得所有Task的好处（传播至、异常、Continuation等）
    不需要在操作时阻塞线程
    初始化一个实例即可
    它有一个Task属性可返回一个Task
    该Task完全由TaskCompletionSource对象控制
    调用任意一个方法都会给Task发信号：
    完成、故障、取消

    这些方法只能调用一次，如果再次调用：
    SetXXX会抛出异常
    TryXXX会返回false */  

    /* 方法如下 */
    public class TaskCompletionSource<TResult>
    {
        public TaskCompletionSource();

        public TaskCompletionSource(object? state);

        public TaskCompletionSource(TaskCreationOptions creationOptions);

        public TaskCompletionSource(object? state, TaskCreationOptions creationOptions);

        public Task<TResult> Task { get; }

        public void SetCanceled();

        public void SetException(IEnumerable<Exception> exceptions);

        public void SetException(Exception exception);

        public void SetResult(TResult result);

        public bool TrySetCanceled();

        public bool TrySetCanceled(CancellationToken cancellationToken);

        public bool TrySetException(IEnumerable<Exception> exceptions);

        public bool TrySetException(Exception exception);

        public bool TrySetResult(TResult result);
    }  


    /*
    *CODE1
    */
    var tcs = new TaskCompletionSource<int>();
    new Thread(() =>
    {
        Thread.Sleep(5000);
        tcs.SetResult(42);
    })
    {
        IsBackground = true
    }.Start();

    Task<int> task = tcs.Task;
    Console.WriteLine(task.Result);

    /*CODE2
    * 调用此方法相当于调用Task.Factory.StartNew
    * 并使用TaskCreationOptions.LongRunning选项来创建非线程池的线程
    */
    Task<TResult> Run<TResult>(Func<TResult> func) 
    {
        var tcs = new TaskCompletionSource<TResult>();
        new Thread(() =>
        {
            try
            {
                tcs.SetResult(func());
            }
            catch (Exception ex)
            {

                tcs.SetException(ex);
            }
        })
        {
            IsBackground = true
        }.Start();
        return tcs.Task;
    }

    /* TaskCompletionSource自身创建Task，但并不占用线程（见示例代码）
    特别需要说明的一点，Task中的Delay和Thread的Sleep不一样的是，Sleep不占用CPU处理资源而Delay会，因为它只是延迟了几秒执行代码而已 */
    static void Main(string[] args)
    {
        //5秒钟之后，Continuation开始的时候，才占用线程
        Delay(5000).GetAwaiter().OnCompleted(() => Console.WriteLine(42));
        Console.ReadKey();
    }

    static Task Delay(int milliseconds) 
    {
        var tcs = new TaskCompletionSource<object>();
        var timer = new System.Timers.Timer(milliseconds) { AutoReset = false };
        timer.Elapsed += delegate { timer.Dispose(); tcs.SetResult(null); };
        timer.Start();
        return tcs.Task;
    }
```

### async/await关键字
>在C#5.0中出现的 async和await ，让异步编程变得和同步编程一样简单

```C#
    class Program
    {
        static void Main(string[] args)
        {
            string content = GetContentAsync(Environment.CurrentDirectory + @"/test.txt").Result;
            //调用同步方法
            //string content = GetContent(Environment.CurrentDirectory + @"/test.txt");
            Console.WriteLine(content);
            Console.ReadKey();
        }
        //异步读取文件内容
        async static Task<string> GetContentAsync(string filename)
        {
            
            FileStream fs = new FileStream(filename, FileMode.Open);
            var bytes = new byte[fs.Length];
            //ReadAync方法异步读取内容，不阻塞线程
            Console.WriteLine("开始读取文件");
            int len = await fs.ReadAsync(bytes, 0, bytes.Length);
            string result = Encoding.UTF8.GetString(bytes);
            return result;
        }
        //同步读取文件内容
        static string GetContent(string filename)
        {
            FileStream fs = new FileStream(filename, FileMode.Open);
            var bytes = new byte[fs.Length];
            //Read方法同步读取内容，阻塞线程
            int len =  fs.Read(bytes, 0, bytes.Length);
            string result = Encoding.UTF8.GetString(bytes);
            return result;
        }
        /* test.txt内容是【hello world！】执行结果为：
        开始读取文件
        hello world! */
    }
```
>上边的例子也写出了同步读取的方式，将main函数中的注释去掉即可同步读取文件内容。我们可以看到异步读取代码和同步读取代码基本一致。async/await让异步编码变得更简单，我们可以像写同步代码一样去写异步代码


#### await方法做了什么
- [剖析C#异步方法](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c/)  

    - 最开始是任务并行库（Task Parallel Library）（TPL），然后C#5引入async/await。
    - 让我们将“异步方法”定义为一个被上下文（contextual）关键字async所标记的方法。这并不意味着这个方法异步地执行。甚至这并不意味着这个方法是异步的。这个关键字的意思只是：编译器会对这个方法进行一些特殊的转换处理。
> “await”关键字告诉编译器在”async”标记的方法中插入一个可能的挂起/唤醒点。  
逻辑上，这意味着当你写”await someObject;”时，编译器将生成代码来检查someObject代表的操作是否已经完成。如果已经完成，则从await标记的唤醒点处继续开始同步执行；如果没有完成，将为等待的someObject生成一个continue委托，当someObject代表的操作完成的时候调用continue委托。这个continue委托将控制权重新返回到”async”方法对应的await唤醒点处。  
返回到await唤醒点处后，不管等待的someObject是否已经经完成，任何结果都可从Task中提取，或者如果someObject操作失败，发生的任何异常随Task一起返回或返回给SynchronizationContext。  
在代码中，意味着当你写：  
await someObject;   
编译器会生成一个包含 MoveNext 方法的状态机类：  
```C#
    /* 我们看看这份异步代码,看完再看编译器为await做了什么 */
    class StockPrices
    {
        private Dictionary<string, decimal> _stockPrices;
        public async Task<decimal> GetStockPriceForAsync(string companyId)
        {
            await InitializeMapIfNeededAsync();
            _stockPrices.TryGetValue(companyId, out var result);
            return result;
        }
    
        private async Task InitializeMapIfNeededAsync()
        {
            if (_stockPrices != null)
                return;
    
            await Task.Delay(42);
            // 从外部数据源或内存中的缓存得到股票价格
            _stockPrices = new Dictionary<string, decimal> { { "MSFT", 42 } };
        }
    }

    /* 编译器为每个异步方法都生成一个状态机，并将原来方法中所有的逻辑移到状态机中 */
    struct _GetStockPriceForAsync_d__1 : IAsyncStateMachine
    {
        public StockPrices __this;
        public string companyId;
        public AsyncTaskMethodBuilder<decimal> __builder;
        public int __state;
        private TaskAwaiter __task1Awaiter;
    
        public void MoveNext()
        {
            decimal result;
            try
            {
                TaskAwaiter awaiter;
                if (__state != 0)
                {
                    // 生成的状态机的状态1：
                    if (string.IsNullOrEmpty(companyId))
                        throw new ArgumentNullException();
    
                    awaiter = __this.InitializeLocalStoreIfNeededAsync().GetAwaiter();
    
                    // 热路径优化：如果任务已经完成，那么状态机自动跳到下一步
                    if (!awaiter.IsCompleted)
                    {
                        __state = 0;
                        __task1Awaiter = awaiter;
    
                        // 下面的调用终究会导致状态机的装箱（boxing）
                        __builder.AwaitUnsafeOnCompleted(ref awaiter, ref this);
                        return;
                    }
                }
                else
                {
                    awaiter = __task1Awaiter;
                    __task1Awaiter = default(TaskAwaiter);
                    __state = -1;
                }
    
                // GetResult返回void，但是如果被等待的任务失败了，它就会抛出异常
                // 这个异常之后会被捕捉并改变“结果任务”。
                awaiter.GetResult();
                __this._stocks.TryGetValue(companyId, out result);
            }
            catch (Exception exception)
            {
                // 最终状态：失败
                __state = -2;
                __builder.SetException(exception);
                return;
            }
    
            // 最终状态：成功
            __state = -2;
            __builder.SetResult(result);
        }
    
        void IAsyncStateMachine.SetStateMachine(IAsyncStateMachine stateMachine)
        {
            __builder.SetStateMachine(stateMachine);
        }
    }
```

#### 什么是awaiter?
>虽然Task和Task<TResult>是两个非常普遍的等待类型(“awaitable”)，但这并不表示只有这两个的等待类型。
“awaitable”可以是任何类型，它必须公开一个GetAwaiter() 方法并且返回有效的”awaiter”。这个GetAwaiter() 可能是一个实例方法（eg:Task或Task<TResult>的实例方法），或者可能是一个扩展方法

>“awaiter”是”awaitable”对象的GetAwaiter()方法返回的符合特定的模式的类型。”awaiter”必须实现System.Runtime.CompilerServices.INotifyCompletion接口（，并可选的实现System.Runtime.CompilerServices.ICriticalNotifyCompletion接口）。除了提供一个INotifyCompletion接口的OnCompleted方法实现（，可选提供ICriticalNotifyCompletion接口的UnsafeCompleted方法实现），还必须提供一个名为IsCompleted的Boolean属性以及一个无参的GetResult()方法。GetResult()返回void，如果”awaitable”代表一个void返回操作，或者它返回一个TResult，那么“awaitable”代表一个TResult返回操作。


- [扩展异步方法](https://devblogs.microsoft.com/premier-developer/extending-the-async-methods-in-c/)

```C#
    public struct LazyAwaiter<T> : INotifyCompletion
    {
        private readonly Lazy<T> _lazy;
    
        public LazyAwaiter(Lazy<T> lazy) => _lazy = lazy;
    
        public T GetResult() => _lazy.Value;
    
        public bool IsCompleted => true;
    
        public void OnCompleted(Action continuation) { }
    }
    
    public static class LazyAwaiterExtensions
    {
        public static LazyAwaiter<T> GetAwaiter<T>(this Lazy<T> lazy)
        {
            return new LazyAwaiter<T>(lazy);
        }
    }

    public static async Task Foo()
    {
        var lazy = new Lazy<int>(() => 42);
        var result = await lazy;
        Console.WriteLine(result);
    }

```

#### await和Task.Wait一样吗？不一样
    >task.Wait()”是一个同步，可能阻塞的调用。它不会立刻返回到Wait()的调用者，直到这个任务进入最终状态，这意味着已进入RanToCompletion，Faulted，或Canceled完成状态。相比之下，”await task;”告诉编译器在”async”标记的方法内部插入一个隐藏的挂起/唤醒点，这样，如果等待的task没有完成，异步方法也会立马返回给调用者，当等待的任务完成时唤醒它从隐藏点处继续执行。当”await task;”会导致比较多应用程序无响应或死锁的情况下使用“task.Wait()”
    - Task.Wait  
    C#里，Task不是专为异步准备的，它表达的是一个线程，是工作在线程池里的一个线程。异步是线程的一种应用，多线程也是线程的一种应用。Wait，以及Status、IsCanceled、IsCompleted、IsFaulted等等，是给多线程准备的方法，跟异步没有半毛钱关系。当然你非要在异步中使用多线程的Wait或其它，从代码编译层面不会出错，但程序会。  
    Task.Wait()是一个同步方法，用于多线程中阻塞等待。  
    <br/>
    - await  
    意思1：在异步中，await表达的意思是：当前线程/方法中，await引导的方法出结果前，跳出当前线程/方法，从调用当前线程/方法的位置，去执行其它可能执行的线程/方法，并在引导的方法出结果后，把运行点拉回到当前位置继续执行；直到遇到下一个await，或线程/方法完成返回，跳回去刚才外部最后执行的位置继续执行。  
    意思2：等异步的运行结果。

    - async  
    在异步编程的规范中，async修饰的方法，仅仅表示这个方法在内部有可能采用异步的方式执行，CPU在执行这个方法时，会放到一个新的线程中执行。那这个方法，最终是否采用异步执行，不决定于是否用await方式调用这个方法，而决定于这个方法内部，是否有await方式的调用

#### task.Result和task.GetAwaiter().GetResult()有区别吗？
>有。但仅仅在任务以非成功状态完成的情况下。如果task是以RanToCompletion状态完成，那么这两个语句是等价的。然而，如果task是以Faulted或Canceled状态完成，task.Result将传播一个或多个异常封装而成的AggregateException对象；而”task.GetAwaiter().GetResult()”将直接传播异常(如果有多个任务，它只会传播其中一个)。  
关于为什么会存在这个差异，请看[《异常处理》](https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/exception-handling-task-parallel-library)

####  “await”是如何关联到当前SynchronizationContext

>这完全取决于被等待的类型。对于给定的”awaitable”，编译器生成的代码最终会调用”awaiter”的OnCompleted()方法，并且传递将执行的continue委托。编译器生成的代码对SynchronizationContext一无所知，仅仅依赖当等待的操作完成时调用OnCompleted()方法时所提供的委托。这就是OnCompleted()方法，它负责确保委托在”正确的地方”被调用，”正确的地方”完全由”awaiter”决定。  

>正在等待的任务(由Task和Task<TResult>的GetAwaiter方法分别返回的TaskAwaiter和TaskAwaiter<TResult>类型)的默认行为是在挂起前捕获当前的SynchronizationContext，然后等待task的完成，如果能捕获到当前的SynchronzationContext，调用continue委托将控制权返回到SynchronizationContext中。所以，例如，如果你在应用程序的UI线程上执行”await task;”，如果当前SynchronizationContext非空则将调用OnCompleted()，并且在任务完成时，将使用UI的SynchronizationContext传播continue委托返回到UI线程。

>当你等待一个任务，如果没有当前SynchronizationContext，那么系统会检查当前的TaskScheduler，如果有，当task完成时将使用TaskScheduler调度continue委托。

>如果SynchronizationContext和TaskScheduler都没有，无法迫使continue委托返回到原来的上下文，或者你使用”await task.ConfigureAwait(false)代替”await task;”，然后continue委托不会迫使返回到原来上下文并且将允许在系统认为合适的地方继续运行。这通常意味着要么以同步方式运行continue委托，无论等待的task在哪完成；要么使用ThreadPool中的线程运行continue委托。

### ET中关于异步编程的介绍
- [简单粗暴的C#的协程](https://github.com/egametang/ET/blob/master/Book/2.1CSharp%E7%9A%84%E5%8D%8F%E7%A8%8B.md)：开个Thread,将阻塞放到该线程中,然后回调通过SynchronizationContext压回主线程中处理。如处理中有阻塞又需要开个Thread...

```C#
    class Program
    {
        private static int loopCount = 0;

        private static void Main()
        {
            OneThreadSynchronizationContext _ = OneThreadSynchronizationContext.Instance;
            
            WaitTimeAsync(5000, WaitTimeFinishCallback);
            
            while (true)
            {
                OneThreadSynchronizationContext.Instance.Update();
                
                Thread.Sleep(1);
                
                ++loopCount;
                if (loopCount % 10000 == 0)
                {
                    Console.WriteLine($"loop count: {loopCount}");
                }
            }
        }

        private static void WaitTimeAsync(int waitTime, Action action)
        {
            Thread thread = new Thread(()=>WaitTime(waitTime, action));
            thread.Start();
        }
        
        private static void WaitTimeFinishCallback()
        {
            Console.WriteLine($"WaitTimeAsync finsih loopCount的值是: {loopCount}");
            WaitTimeAsync(4000, WaitTimeFinishCallback3);
        }

        /// <summary>
        /// 在另外的线程等待
        /// </summary>
        private static void WaitTime(int waitTime, Action action)
        {
            Thread.Sleep(waitTime);
            
            // 将action扔回主线程执行
            OneThreadSynchronizationContext.Instance.Post((o)=>action(), null);
        }

        /* 下面两个方法是因为action里面又需要阻塞 */
        private static void WaitTimeFinishCallback3()
        {
            Console.WriteLine($"WaitTimeAsync finsih loopCount的值是: {loopCount}");
            WaitTimeAsync(3000, WaitTimeFinishCallback2);
        }
        
        private static void WaitTimeFinishCallback2()
        {
            Console.WriteLine($"WaitTimeAsync finsih loopCount的值是: {loopCount}");
        }
    }

```
- [C#的更好协程](https://github.com/egametang/ET/blob/master/Book/2.2%E6%9B%B4%E5%A5%BD%E7%9A%84%E5%8D%8F%E7%A8%8B.md) :C#帮我们把上面简单粗暴的回调形式改成同步的形式，返回Task，用await等待结果

```C#
    class Program
    {
        private static int loopCount = 0;
        
        static void Main(string[] args)
        {
            SynchronizationContext.SetSynchronizationContext(OneThreadSynchronizationContext.Instance);

            Console.WriteLine($"主线程: {Thread.CurrentThread.ManagedThreadId}");
            
            Crontine();
            
            while (true)
            {
                OneThreadSynchronizationContext.Instance.Update();
                
                Thread.Sleep(1);
                
                ++loopCount;
                if (loopCount % 10000 == 0)
                {
                    Console.WriteLine($"loop count: {loopCount}");
                }
            }
        }

        private static async void Crontine()
        {
            await WaitTimeAsync(5000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
            await WaitTimeAsync(4000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
            await WaitTimeAsync(3000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
        }
        
        /* 利用了TaskCompletionSource类替代了之前传入的Action参数，WaitTimeAsync方法返回了一个Task类型的结果。WaitTime中我们把action()替换成了tcs.SetResult(true),WaitTimeAsync方法前使用await关键字，这样可以将一连串的回调改成同步的形式。
        （虽然...但对熟悉了Nodejs的人来说看了这两个方法就会觉得莫名其妙,完全就没有感觉有多简洁） */
        private static Task WaitTimeAsync(int waitTime)
        {
            TaskCompletionSource<bool> tcs = new TaskCompletionSource<bool>();
            Thread thread = new Thread(()=>WaitTime(waitTime, tcs));
            thread.Start();
            return tcs.Task;
        }
        
        /// <summary>
        /// 在另外的线程等待
        /// </summary>
        private static void WaitTime(int waitTime, TaskCompletionSource<bool> tcs)
        {
            Thread.Sleep(waitTime);
            
            // 将tcs扔回主线程执行
            tcs.SetResult(true);
        }
    }
```
- [单线程异步编程](https://github.com/egametang/ET/blob/master/Book/2.3%E5%8D%95%E7%BA%BF%E7%A8%8B%E5%BC%82%E6%AD%A5.md):我们在之前的例子中使用了Sleep来实现时间的等待，每一个计时器都需要使用一个线程，会导致线程切换频繁，这个实现效率很低，平常是不会这样做的。一般游戏逻辑中会设计一个单线程的计时器，我们这里做一个简单的实现，用来讲解单线程异步。
```C#
    class Program
    {
        private static int loopCount = 0;

        private static long time;
        private static TaskCompletionSource<bool> tcs;
        
        static void Main(string[] args)
        {
            Console.WriteLine($"主线程: {Thread.CurrentThread.ManagedThreadId}");

            Crontine();
            
            while (true)
            {
                Thread.Sleep(1);

                CheckTimerOut();
                
                ++loopCount;
                if (loopCount % 10000 == 0)
                {
                    Console.WriteLine($"loop count: {loopCount}");
                }
            }
        }
        
        private static async void Crontine()
        {
            await WaitTimeAsync(5000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
            await WaitTimeAsync(4000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
            await WaitTimeAsync(3000);
            Console.WriteLine($"当前线程: {Thread.CurrentThread.ManagedThreadId}, WaitTimeAsync finsih loopCount的值是: {loopCount}");
        }

        private static void CheckTimerOut()
        {
            if (time == 0)
            {
                return;
            }
            long nowTicks = DateTime.Now.Ticks / 10000;
            if (time > nowTicks)
            {
                return;
            }

            time = 0;
            tcs.SetResult(true);
        }
        
        private static Task WaitTimeAsync(int waitTime)
        {
            TaskCompletionSource<bool> t = new TaskCompletionSource<bool>();
            time = DateTime.Now.Ticks / 10000 + waitTime;
            tcs = t;
            return t.Task;
        }
    }
```



### ETTASK   
> ET的异步核心在于ETTask,我们看看ETTask做了什么  
> 看ETTask能够理解Task的实现。



### Geek
Geek的异步。
    


--- 
# Actor

> 以前写服务端时，为了进程间通信，很简单粗暴地使用shm作为共享内存然后加锁。  
> 最近看C#服务端，发现无论是多进程还是多线程，都会有Actor模型。  
> 看概念很容易理解，就是消息通信而已。但看一些框架却很绕很难理解。

<br/>

### 网上关于Actor的解释

>传统的编程模型通常使用回调和同步对象（如锁）来协调任务和访问共享数据，从宏观看传统模型：任务是一步步紧接着完成的，资源是需要抢占的。   
Actor模式是一种并发模型，与共享内存完全相反，Actor模型share nothing。所有的线程(或进程)通过消息传递的方式进行合作，这些线程(或进程)称为Actor， 预先定义了任务的流水线后，不关注数据什么时候流到这个任务 ，专注完成工序任务。  
Actor 是一种可以响应和发送消息的独立实体，某种程度上，它就像是在另一个进程中运行属于它自己的程序。

<br/>

>Actor模型描述了一组为避免并发编程的公理：  
所有的Actor状态是本地的，外部是无法访问的。  
Actor必须通过消息传递进行通信。  
一个Actor可以响应消息、退出新Actor、改变内部状态、将消息发送到一个或多个Actor。  
Actor可能会堵塞自己但Actor不应该堵塞自己运行的线程。  

### 所以Actor到底做什么？

<br/>

### C#怎么用[TPL DataFlow](https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/dataflow-task-parallel-library)构建Actor？
<!-- - TPL Dataflow 核心概念 
    - Block
        - Buffering Only    【Buffer不是缓存Cache的概念， 而是一个缓冲区的概念】
        - Execution
        - Grouping
    - Execution Block  可执行的块有两个核心组件
        - 输入、输出消息的缓冲区（一般称为Input,Output队列）
        - 在消息上执行动作的委托  -->

- 先看个[例子](https://blog.jayway.com/2013/11/15/an-actor-model-implementation-in-c-using-tpl-dataflow/#comment-463337)  
    我们是一家银行，我们有账户。每个账户都有余额。我们希望能够存入资金并检查余额  

```C#
    /* Messages 消息的基本形式是发送给参与者的“某物” */
    public abstract class Message {
    }

    /* 要存钱我们会发送带有金额的存款消息 */
    public class Deposit : Message{
        public decimal Amount { get; set; }
    }
    /* 要检查余额，我们发送一条 QueryBalance 消息。  
       查询的结果应该作为消息发送给另一个参与者，因此我们将接收者包含在查询消息中。 */
    public class QueryBalance : Message{
        public Actor Receiver { get; set; }
    }
    /* 最后的消息是余额检查的结果 */
    public class Balance : Message{
        public decimal Amount { get; set; }
    }
    /* 我们使用DataFlow 库中的ActionBlock 来实现一个actor。  
      ActionBlock有一个输入队列，您可以将消息发布到该队列，以及一个将为每个接收到的消息调用的操作。 */
    public abstract class Actor{
        private readonly ActionBlock<Message> action;

        public Actor(){
            /* 在 Actor 的构造函数中，我们创建一个将 Actor 和消息转换为动态对象的动作，然后调用 Handle 方法。  
            这意味着运行时将使用消息类型的参数查找名为“Handle”的方法并调用它。 */
            action = new ActionBlock<Message>(message =>
            {
                dynamic self = this;
                dynamic mess = message;
                self.Handle(mess);
            });
        }

        public void Send(Message message){
            action.Post(message);
        }
    }
    /* 账户参与者应该有余额并处理存款和查询余额消息 */
    public class AccountActor : Actor {
        private decimal balance;

        public void Handle(Deposit message){
            balance += message.Amount;
        }

        public void Handle(QueryBalance message){
            message.Receiver.Send(new Balance {Amount = balance});
        }
    }
    /* 我们还需要一个输出余额查询结果的actor */
    public class OutputActor : Actor {
        public void Handle(Balance message){
            Console.WriteLine("Balance is {0}", message.Amount);
        }
    }
    /* 现在我们可以创建Actor并向他们发送消息。 */
    public class Program {
        public static void Main(string[] args){
            var account = new AccountActor();
            var output = new OutputActor();

            account.Send(new Deposit {Amount = 50});
            account.Send(new QueryBalance {Receiver = output});

            Console.WriteLine("Done!");
            Console.ReadLine();
        }
    }
    /* 运行结果 */
    // Done!
    // Balance is 50

    /*   
    请注意“完成！” 是在余额表之前！程序发送消息并在Actor处理完消息之前完成。我们如何知道参与者何时处理完所有消息？  
    为了解决这个问题，我们使用了 DataFlow 的完成特性，并在 Actor 类中添加了一个 Completion 属性  
    这告诉 ActionBlock 停止接收消息并返回一个任务，该任务将在当前队列中的所有消息都被处理后完成  
    */
    public abstract class Actor{
        private readonly ActionBlock<Message> action;
        public Actor(){
            action = new ActionBlock<Message>(message =>
            {
                dynamic self = this;
                dynamic mess = message;
                self.Handle(mess);
            });
        }
        public void Send(Message message){
            action.Post(message);
        }

        public Task Completion{
            get{
                action.Complete();
                return action.Completion;
            }
        }
    }

    public class Program{
        public static void Main(string[] args)
        {
            var account = new AccountActor();
            var output = new OutputActor();

            account.Send(new Deposit {Amount = 50});
            account.Send(new QueryBalance {Receiver = output});

            account.Completion.Wait();
            output.Completion.Wait();

            Console.WriteLine("Done!");
            Console.ReadLine();
        }
    }
    /* 运行结果 */
    // Balance is 50
    // Done!   
```

- Dataflow官方解释

    数据流编程模型与消息传递这一概念相关，其中程序的独立组件通过发送消息相互通信。 在应用组件间传播消息的一种方法是，调用 Post 和 DataflowBlock.SendAsync 方法，向目标数据流块发送消息（Post 同步运行，SendAsync 异步运行），再调用 Receive、ReceiveAsync 和 TryReceive 方法接收源数据流块发送的消息。 您可以通过向头节点（目标块）发送输入数据，从管道的终端节点或网络的终端节点（一个或多个源块）接收输出数据来使用数据流管道或网络组合使用这些方法。 您还可以使用 Choose 方法从提供的第一个拥有可用数据的源读取数据，并对该数据执行操作。

    源数据流块通过调用方法 ITargetBlock<TInput>.OfferMessage 向目标数据流块提供数据。 目标块通过以下三种方式之一来回应提供的消息：它可以接受消息，拒绝消息或推迟消息。 当目标接受消息时，OfferMessage 方法会返回 Accepted。 当目标拒绝消息时，OfferMessage 方法会返回 Declined。 当目标要求它不再接收来自源的任何消息时，OfferMessage 会返回 DecliningPermanently。 预定义的源块类型在这些返回值接收后不会向链接的目标提供消息，并且它们会自动取消这些目标的链接。

    当目标块推迟消息以备日后使用时，OfferMessage 方法会返回 Postponed。 推迟消息的目标块可以稍后调用 ISourceBlock<TOutput>.ReserveMessage 方法，以尝试暂留所提供的消息。 此时，消息仍可用，并且可由该目标块使用，否则表明该消息已由另一个目标接收。 如果目标数据流块稍后需要消息或不再需要消息，它会分别调用 ISourceBlock<TOutput>.ConsumeMessage 或 ReleaseReservation 方法。 消息预留通常由以非贪婪模式运行的数据流块类型使用。 非贪婪模式将在本文档的后面详细介绍。 除了保留推迟的消息，目标块也可以使用 ISourceBlock<TOutput>.ConsumeMessage 方法来尝试直接使用推迟的消息。

    ActionBlock是在单线程上执行的，出于性能原因希望使用多个专用线程来处理时，只要设置MaxDegreeOfParallelism来配置线程数量即可。

- Dataflow学习：[具有TPL数据流和故障处理的C#作业队列](https://michaelscodingspot.com/c-job-queues-part-3-with-tpl-dataflow-and-failure-handling/)
  

### Actor的实现是做了什么？

<br/>


- Geek中Actor
    1. GeekServer.Core/Actor
        - ActorAttribute.cs 
            - NotAwait
            - ThreadSafe
            - LongTimeTake
            - CanBeCalledBySameComp
            - ExecuteTime  
        这些特性在AgentGenerator生成代码时使用
        - BaseActor.cs
            - ActionBlock<WorkWrapper> actionBlock;   &emsp;这里看WorkWrapper是Message  
            - InnerRun()  &emsp;其中调用了wrapper.DoTask()，类似例子中的Handle 
        - WorkerActor.cs  &emsp;继承自BaseActor，某种参与者Actor
        - WorkWrapper.cs  &emsp;类似例子中的Message,还有ActionWrapper、FuncWrapper、ActionAsyncWrapper、FuncAsyncWrapper
            - SetContext()&emsp;为何DoTask时要SetContext()?用于环路调用检测，判断是否需要入队。  
> 这里理解下Geek的Actor的作用。  
> 在服务器启动时将Component注册到Actor的作用似乎没看到？  
> 

    ---
    2. UnityDemo/Assets/Scripts/Framework/Actor
        - Actor.cs
        - UniActor.cs
        - WorkWrapper.cs

- ET的Actor
    - ActorLocation

    >基于Actor开发的架构，在给某个Actor发送消息时必须要知道Actor对象的地址，一旦Actor地址改变了（玩家进行了跨服，逻辑上就是用户Actor单位跳转了进程），那么消息就无法准确到达。为了解决这一现象，ActorLocation模型诞生。  
    ActorLocation相比Actor多了一个用于注册Actor位置的第三方ActorLocationSever，所有Actor消息的发送都要经由第三方转发。在普通Actor改变地址时（进行跨服跳转）必须注册上报Actor新的地址。由于Actor的唯一ID是不会改变的，那么后续发送的消息可以更具这个唯一ID找到Actor对应的新地址，然后转发。


--- 

# 序列化工具

### MessagePack使用

```C#
    // 标记 MessagePackObjectAttribute
    [MessagePackObject]
    public class MyClass
    {
        // Key 是序列化索引，对于版本控制非常重要。
        [Key(0)]
        public int Age { get; set; }

        [Key(1)]
        public string FirstName { get; set; }

        [Key(2)]
        public string LastName { get; set; }

        // 公共成员中不序列化目标，标记IgnoreMemberAttribute
        [IgnoreMember]
        public string FullName { get { return FirstName + LastName; } }
    }

    /* 序列化 */    
    class Program
    {
        static void Main(string[] args)
        {
            var mc = new MyClass
            {
                Age = 99,
                FirstName = "hoge",
                LastName = "huga",
            };

            // 序列化
            byte[] bytes = MessagePackSerializer.Serialize(mc);
            //反序列化
            MyClass mc2 = MessagePackSerializer.Deserialize<MyClass>(bytes);

            // 你可以将msgpack二进制转储为可读的json。
            // 在默认情况下，MeesagePack for C＃减少了属性名称信息。
            // [99,"hoge","huga"]
            string json = MessagePackSerializer.ToJson(bytes);
            Console.WriteLine(json);

            Console.ReadKey();

        }
    }
```

### protobuf-net使用


# Net

### Microsoft.AspNetCore.Builder.WebApplication

> 用来创建ASP.NET Core 中的 Kestrel Web 服务器  

> 为何不能直接用Kestrel,需要nginx等反向代理服务器。  
> 1是出于安全的目的，需要nginx处理超时、大小限制、并发连接等问题。  
> 2是Kestrel不支持多进程间共享端口。反向代理可以让多个应用共享该端口。

> Kestrel是一款基于中间件来处理tcp连接的服务器，并内置了http(包含websocket、SignalR)解析中间件。也就是说，我们完全可以给kestrel添加其它中间件，用来处理非http的连接的业务场景，让kestrel使用一个端口支持多种协议或多协议一个端口一种协议的要求。  

>[Kestrel创建中间件的例子](https://www.cnblogs.com/kewei/p/12775469.html)

```C#

```
### Microsoft.AspNetCore.Mvc.Cors
> 浏览器安全性可防止网页向不处理网页的域发送请求。 此限制称为同域策略。 同域策略可防止恶意站点从另一站点读取敏感数据。 有时，你可能希望允许其他网站向自己的应用发出跨源请求,用Cors就可以实现。

```C#
    builder.Services.AddCors(options =>
    {
        options.AddPolicy
            (name: "myCors",
                builde =>
                {
                    builde.WithOrigins("*", "*","*")
                    .AllowAnyOrigin()
                    .AllowAnyHeader()
                    .AllowAnyMethod();
                }
            );
    });

    /* 最后app处再加上 */
    app.UseCors("myCors");
```

### System.Net.Http.HttpClient

> 能够同时在客户端与服务端同时使用的 HTTP 组件（比如处理 HTTP 标头和消息）， 为客户端和服务端提供一致的编程模型。  
> 可以用在服务器端向钉钉等工具发送错误通知。

```C#
namespace ClassLibrary1
{
    public class Class1
    {
        /// <summary>
        /// 登录博客园
        /// </summary>
        public static void LoginCnblogs()
        {
            HttpClient httpClient = new HttpClient();
            httpClient.MaxResponseContentBufferSize = 256000;
            httpClient.DefaultRequestHeaders.Add("user-agent", "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36");
            String url = "http://passport.cnblogs.com/login.aspx";
            HttpResponseMessage response = httpClient.GetAsync(new Uri(url)).Result;
            String result = response.Content.ReadAsStringAsync().Result;

            String username = "hi_amos";
            String password = "密码";

            do
            {
                String __EVENTVALIDATION = new Regex("id=\"__EVENTVALIDATION\" value=\"(.*?)\"").Match(result).Groups[1].Value;
                String __VIEWSTATE = new Regex("id=\"__VIEWSTATE\" value=\"(.*?)\"").Match(result).Groups[1].Value;
                String LBD_VCID_c_login_logincaptcha = new Regex("id=\"LBD_VCID_c_login_logincaptcha\" value=\"(.*?)\"").Match(result).Groups[1].Value;

                //图片验证码
                url = "http://passport.cnblogs.com" + new Regex("id=\"c_login_logincaptcha_CaptchaImage\" src=\"(.*?)\"").Match(result).Groups[1].Value;
                response = httpClient.GetAsync(new Uri(url)).Result;
                Write("amosli.png", response.Content.ReadAsByteArrayAsync().Result);
                
                Console.WriteLine("输入图片验证码：");
                String imgCode = "wupve";//验证码写到本地了，需要手动填写
                imgCode =  Console.ReadLine();

                //开始登录
                url = "http://passport.cnblogs.com/login.aspx";
                List<KeyValuePair<String, String>> paramList = new List<KeyValuePair<String, String>>();
                paramList.Add(new KeyValuePair<string, string>("__EVENTTARGET", ""));
                paramList.Add(new KeyValuePair<string, string>("__EVENTARGUMENT", ""));
                paramList.Add(new KeyValuePair<string, string>("__VIEWSTATE", __VIEWSTATE));
                paramList.Add(new KeyValuePair<string, string>("__EVENTVALIDATION", __EVENTVALIDATION));
                paramList.Add(new KeyValuePair<string, string>("tbUserName", username));
                paramList.Add(new KeyValuePair<string, string>("tbPassword", password));
                paramList.Add(new KeyValuePair<string, string>("LBD_VCID_c_login_logincaptcha", LBD_VCID_c_login_logincaptcha));
                paramList.Add(new KeyValuePair<string, string>("LBD_BackWorkaround_c_login_logincaptcha", "1"));
                paramList.Add(new KeyValuePair<string, string>("CaptchaCodeTextBox", imgCode));
                paramList.Add(new KeyValuePair<string, string>("btnLogin", "登  录"));
                paramList.Add(new KeyValuePair<string, string>("txtReturnUrl", "http://home.cnblogs.com/"));
                response = httpClient.PostAsync(new Uri(url), new FormUrlEncodedContent(paramList)).Result;
                result = response.Content.ReadAsStringAsync().Result;
                Write("myCnblogs.html",result);
            } while (result.Contains("验证码错误，麻烦您重新输入"));

            Console.WriteLine("登录成功！");
            
            //用完要记得释放
            httpClient.Dispose();
        }

        public static void Main()
        {
              LoginCnblogs();
        }

}
```

