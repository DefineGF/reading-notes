

### Task

*   默认使用线程池，即后台线程。主线程结束时，所有tasks都结束；
*   Task.Run 返回一个Task对象，用以监视其过程；
*   通过Task.Status 属性来跟踪Task 的执行状态；

##### 创建 Task

```c#
private static void CreateTask()
{
    // 方式1
    var task = Task.Run(() =>
    {
		Console.WriteLine("create_task");
	});
	task.Wait();  // 阻塞主线程; task 执行完毕后才执行下方语句
    
    // 方式2
    var task2 = new Task(() => {
        //
    });
    task2.Wait();
    
    // 方式3
    var task3 = Task.Factory.StartNew(() => {
        //
    });
    task3.Wait();
    
	Console.WriteLine("运行完毕!");
}

```

创建长时间任务：

```c#
private static void CreateLongTask()
{
    Task task = Task.Factory.StartNew(() =>
    {
        Console.WriteLine("run the long task!");
        Thread.Sleep(1000);
    }, TaskCreationOptions.LongRunning);
    task.Wait();
    Console.WriteLine("long task finished!");
}
```

如果同时运行多个LongRunning  Tasks（尤其是有阻塞状态的），性能将会受到很大影响：

*   如果是IO-Bound：TaskCompletionSource和异步函数可以使用回调代替线程实现并发；
*   如果是Compute-Bound：生产者-消费者队列可对任务并发性限流，避免把其他线程的CPU占尽；

##### Status

```c#
public enum TaskStatus
{
    //
    // 摘要:
    //     The task has been initialized but has not yet been scheduled.
    Created = 0,
    //
    // 摘要:
    //     The task is waiting to be activated and scheduled internally by the .NET Framework
    //     infrastructure.
    WaitingForActivation = 1,
    //
    // 摘要:
    //     The task has been scheduled for execution but has not yet begun executing.
    WaitingToRun = 2,
    //
    // 摘要:
    //     The task is running but has not yet completed.
    Running = 3,
    //
    // 摘要:
    //     The task has finished executing and is implicitly waiting for attached child
    //     tasks to complete.
    WaitingForChildrenToComplete = 4,
    //
    // 摘要:
    //     The task completed execution successfully.
    RanToCompletion = 5,
    //
    // 摘要:
    //     The task acknowledged cancellation by throwing an OperationCanceledException
    //     with its own CancellationToken while the token was in signaled state, or the
    //     task's CancellationToken was already signaled before the task started executing.
    //     For more information, see Task Cancellation.
    Canceled = 6,
    //
    // 摘要:
    //     The task completed due to an unhandled exception.
    Faulted = 7
}

if (task.Status == TaskStatus.RanToCompletion)
{
    //当当前线程状态表示完成时则执行后续操作
    Console.WriteLine("do it");
}
```

##### 返回值

```c#
private static void ReturnTask()
{
	Task<string> task = Task.Run(() =>
    {
    	Console.WriteLine("返回结果的任务!");
        Thread.Sleep(2000);
        return "我滴任务完成了";
    });
	Console.WriteLine("主线程ing");
	task.Wait();
	var result = task.Result;
	Console.WriteLine("获取返回结果: result = " + result);
}
```

执行结果：

> 主线程ing
> 返回结果的任务!
> 获取返回结果: result = 我滴任务完成了

##### Continuation

```c#
private static void ContinueTask()
{
	Task<int> _task = Task.Run(() =>
	{
		Console.WriteLine("do it");
		Thread.Sleep(2000);
		return 1024;
	});
	var waiter = _task.GetAwaiter();
    Console.WriteLine("获取Waiter，当前是否结束：" + _task.IsCompleted); 
    waiter.OnCompleted(() =>
    {
        int result = waiter.GetResult();
        Console.WriteLine("计算完毕，获得结果为: " + result);
    });
    Console.WriteLine("主线程ing");
    _task.Wait();
    Console.WriteLine("函数结束!");
 }
```

运行结果：

> 获取Waiter，当前是否结束：False
> do it
> 主线程ing
> 函数结束!
> 计算完毕，获得结果为: 1024

通过OnCompleted监听Task结束事件；awaiter有以下方法：

*   GetResult：获取返回结果；
*   IsCompleted：是否结束；
*   OnCompleted( () => {})：事件结束时调用；

##### ContinueWith

```c
_mainWorkTask.ContinueWith(t => OnContibuteWork(t));
private void OnContibuteWork(Task t)
{
	Console.WriteLine($"task: id = {Task.CurrentId}, status = {t.Status}, id2 = {t.Id}"); 
	// 2 RanToCompletion 1
}
```

通过输出id可知, ContinueWith启动的Action是通过新的线程进行的,传入的t参数, 即是已经运行完毕的原task

##### 等待任务

*   Task.Wait()：无条件等待直到任务完成；还可以 Wait(Int32) 和 Wait(TimeSpan) 阻止调用线程;
*   Task.WaitAny(Task\[])：等待任何一项完成；
*   Task.WaitAll(Task\[])：等待所有任务完成；

##### 任务取消

可通过 `CancelationTokenSource`和`CancellationToken`来实现这一点。

如果任务取消了，Wait() 和 Result属性都抛出包含OperationCanceledException的AggregateException(实际上是TaskCanceledException， 继承自OperationCanceledException)；状态变为Cancelled，而不是Faulted。

```c#
CancellationTokenSource source = new CancellationTokenSource();
int index = 0;
//开启一个task执行任务
Task task7 = new Task(() =>
{
	while (!source.IsCancellationRequested)
	{
		Thread.Sleep(1000);
		Console.WriteLine($"第{++index}次执行，线程运行中...");
	}
});
task7.Start();

Thread.Sleep(5000);
//source.Cancel()方法请求取消任务，IsCancellationRequested会变成true
source.Cancel();
```

当然也可以用来注册取消事件：

```c#
source.Token.Register(() =>
{
	Console.WriteLine("任务被取消后执行xx操作！");
});
// task
task8.Start();
//延时取消，效果等同于Thread.Sleep(5000);source.Cancel();
source1.CancelAfter(5000);
```

##### 异常

*   异步失败：Status 变为 Faulted（并且IsFaulted 返回 true）
*   Exception属性返回 AggregateException，包含（可能多个）造成任务失败的异常；如果没有，则返回null；
*   如果任务的最终状态为错误，则Wait方法抛出一个AggregateException；

处理多个异常：

```c#
private async void HandleAggreateException()
{
    Task taskResult = null;
    try
    {
        Task t1 = ThrowAfter(1000, "ex1");
        Task t2 = ThrowAfter(2000, "ex2");
        Task t3 = ThrowAfter(3000, "ex3");
        await (taskResult = Task.WhenAll(t1, t2, t3));
    }
    catch (Exception ex)
    {
        Console.WriteLine("ex_msg: " + ex.Message);
        foreach (Exception e in taskResult.Exception.InnerExceptions)
        {
            Console.WriteLine("inner_ex: " + e.Message);
        }
    }
}

private async Task ThrowAfter(int ms, string message)
{
    await Task.Delay(ms);
    Console.WriteLine("抛出异常!");
    throw new Exception(message);
}
```

> 抛出异常!
> 抛出异常!
> 抛出异常!
> ex\_msg: ex1
> inner\_ex: ex1
> inner\_ex: ex2
> inner\_ex: ex3

##### cancel 实例

```c#
CancellationTokenSource source = new CancellationTokenSource();
Task _mainWork = null;

private void TestCancel()
{
    CancellationToken token = source.Token;
    _mainWork = Task.Factory.StartNew(() =>
    {
        for (int i = 0; i < 10; i++)
        {
            Console.WriteLine("working, i = " + i);
            Thread.Sleep(1000);
            if(token.IsCancellationRequested)
            {
                Console.WriteLine("work: 取消!, status = " + _mainWork.Status);
                token.ThrowIfCancellationRequested();
            }
        }
    }, token);
    Console.WriteLine("创建完成!");
    Task.Factory.StartNew(() =>
    {
        Thread.Sleep(3000);
        source.Cancel();
        Console.WriteLine("取消");
    });

    try
    {
        _mainWork.Wait(); // cancel() -> ThrowIfCancellationRequested() 抛出异常
    } 
    catch (AggregateException ae)
    {
        foreach (Exception e in ae.InnerExceptions)
        {
            if (e is TaskCanceledException)
                Console.WriteLine("TaskCanceledException : {0}, 任务状态: {1}", ((TaskCanceledException)e).Message, _mainWork.Status);
            else
                Console.WriteLine("Exception 2 : " + e.GetType().Name);
        }
    }
}
```

