#### Async和Await

async：声明异步方法时使用的修饰符；await：负责消费异步操作；

##### async

声明异步方法：

```c#
public async Task<int> FooAsync() {...}
```

异步方法的返回类型：

* void：为了和事件处理程序兼容：

  ```c#
  // UI 程序
  private async void LoadPrice(object sender, EventArgs e) {
  	priceDisplay.Text = price.ToString("c");
  }
  ```

* Task：不需要返回值；

* Task\<IResult>

##### await

*   告知是否已经完成；
*   如未完成可附加后续操作；
*   获取结果，该结果可能为返回值，但至少可以指明成功或者失败；

##### 工作原理

```c#
var result = await expression;
statement(s);
```

编译器将上述代码转换为：

```c#
var awaiter = expression.GetAwaiter();
awaiter.OnCompleted(() => {
	var result = awaiter.GetResult();
	statement(s);
});
```

##### 实例1

```c#
namespace MultiThread
{
    class AsyncAwait
    {
        static void Main(string[] args)
        {
            Console.WriteLine("before invoke");
            GetNumberTest();
            Console.WriteLine("after invoke");
  
            Thread.Sleep(3000);  					 // 主线程等待子线程
        }

        private static async void GetNumberTest()   // await 不能单独使用
        {
            int number = await GetNumber();         // GetNumber() 返回值是 async Task<int>，
            										// 因此不能使用 int number = 	GetNumber();
            Console.WriteLine("获取返回结果: " + number);
        }
        
        private static async Task<int> GetNumber()  // async Task<int> 返回值是 int 的异步Task
        {
            Task<int> task = Task.Run(() =>
            {
                Thread.Sleep(3000);
                Console.WriteLine("返回数字!");
                return 1024;
            });
            Console.WriteLine("task started~");    // 此时尚未 await，因此可以同步执行此行 输出
            int result = await task;			   // 阻塞进程
            Console.WriteLine("get result!");
            return result;
        }
}
```

运行结果：（需要注意当前线程）

> before invoke                 -> 线程1
>
> task started\~                  -> 线程1
>
> after invoke                    -> 线程1
>
> 返回数字!                         -> 线程4
> get result!                       -> 线程4
> 获取返回结果: 1024       -> 线程4

##### 实例2

```c#
public async Task<int> GetUrlContentLengthAsync()
{
    var client = new HttpClient();
    Task<string> getStringTask = client.GetStringAsync("https://docs.microsoft.com/dotnet");
    
    DoIndependentWork();

    string contents = await getStringTask;
    return contents.Length;
}

void DoIndependentWork()
{
    Console.WriteLine("Working...");
}
// 调用
async void Call () {
    int ans = await GetUrlContentLengthAsync();
}
```

执行过程：

1.  创建HttpClient实例并调用GetStringAsync异步方法下载网站内容；返回Task，将获取的结果保存至getStringTask中；
2.  GetStringAsync将控制权交给GetUrlContentLengthAsync 的调用方；
3.  继续执行同步方法：DoIndependentWork() ；
4.  使用await挂起进度，并把控制权交给调用方；同时将Task\<int> 返回；调用方可以执行不依赖于GetURLContentLengthAsync结果的方法；
5.  `GetStringAsync` 完成并生成一个字符串结果；字符串结果保存在getStringTask中，并有await取出放置在contents内；

##### UI线程与非UI线程

非UI线程：

```c#
private static async void AwaitThread()
{
    Console.WriteLine($"thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId}");

    await Task.Run(() =>
    {
        Console.WriteLine($"in_first_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId}");
        Task.Delay(1000).Wait();
    });

    Console.WriteLine($"out_first_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId}");

    await Task.Run(() =>
    {
        Console.WriteLine($"in_second_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId}");
        Task.Delay(1000).Wait();
    });

    Console.WriteLine($"out_second_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId}");
}
```

运行结果：

> thread\_id: 1, is\_thread\_pool: False, task\_id:
> in\_first\_await! thread\_id: 4, is\_thread\_pool: True, task\_id: 1
> out\_first\_await! thread\_id: 4, is\_thread\_pool: True, task\_id:
> in\_second\_await! thread\_id: 5, is\_thread\_pool: True, task\_id: 4
> out\_second\_await! thread\_id: 5, is\_thread\_pool: True, task\_id:

可以看出：无论是in await还是 out await，await 之间的 代码均不是在 主线程（Thread\_1）中执行；

UI线程：

```c#
async void LogMsg()
{
    message += ($"thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId} \r\n");
    
    await Task.Run(() =>
    {
        message += ($"in_first_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId} \r\n");
    });
    
    message += ($"out_first_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId} \r\n");
    
    await Task.Run(() =>
    {
        message += ($"in_second_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId} \r\n");
    });

    message += ($"out_second_await! thread_id: {Thread.CurrentThread.ManagedThreadId}, " +
        $"is_thread_pool: {Thread.CurrentThread.IsThreadPoolThread}, task_id: {Task.CurrentId} \r\n");

    tb_content.Text = message;
}
```

运行结果：

 <img src />

可以看出，除了使用Task.Run() 使用新的线程之外，其他的工作进行均在主线程（UI线程）进行！

##### WPF 实例

点击按钮启动耗时线程执行操作返回字符串, 然后控件显示字符串内容;
有以下实现方式:

1.  将耗时线程和修改控件操作高度耦合: 即在获取字符串线程内部添加后续控件修改操作;
2.  同步实现: 即顺序执行封装好的耗时线程与控件修改操作;

同步实现:

```c#
private async void LoadContent_Click(object sender, RoutedEventArgs e)
{
    Console.WriteLine("start: ");
    string content = await GetTabelContentAsync();
    _ = this.Dispatcher.BeginInvoke(new Action(() =>
    {
        this.tbContent.Text = content;
    }));
    Console.WriteLine("end!");
}

private async Task<string> GetTabelContentAsync()
{
    Task<string> main_task = Task.Factory.StartNew(() =>
    {
       for (int i = 0; i < 3; i++)
       {
           Console.WriteLine("请等待...");
           Thread.Sleep(1000);
       }
       return "<html><title>this is title!</title></html";
    });
    return await main_task;
}
```

这里同步执行:

> start
> GetTableContentAsync()
> 修改控件
> end

这就意味着LoadContent\_Click需要在耗时线程结束之后, 函数才算完成.

异步实现:

```c#
private  void LoadContent_Click(object sender, RoutedEventArgs e)
{
    Console.WriteLine("start: ");
    GetTabelContentCaller();
    Console.WriteLine("end");
}

private async void GetTabelContentCaller()
{
    string content = await GetTabelContentAsync();
    _ = this.Dispatcher.BeginInvoke(new Action(() =>
      {
          this.tbContent.Text = content;
      }));
}
```

执行顺序:

> start
> end | GetTabelContentCaller

这样 GetTabelContentAsync() 则不会阻塞 Click 方法!

思考一下， GetTabelContentCaller只是将原先Click事件中的代码封装到一个函数中罢了。 为什么可以不阻塞Click事件。

试想一个这样的设计哲学：

*   假如一个异步方法有返回值，调用者用到其返回值。那么调用者方法需要使用 async - await 来阻塞这个异步方法以获取这个异步方法的结果；
*   假如调用者并不需要一个异步方法的返回值, 那么也就无需使用await来阻塞这个异步方法. 那么也就无需调用者与此方法同步了.

分析上述代码:

```c
private async void GetTabelContentCaller()
{
    string content = await GetTabelContentAsync();
    _ = this.Dispatcher.BeginInvoke(new Action(() =>
      {
          this.tbContent.Text = content;
      }));
}
```

执行到 await时, 将异步方法 GetTabelContentAsync 的结果承诺给 content， 之后交由content处理。GetTabelContentCaller 同时完成使命。

因此在Click事件中， 无需同步等待 GetTabelContentCaller 内部执行完毕。