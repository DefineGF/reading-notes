#### ConfigureAwait



##### 非UI场景

默认

```c#
namespace MultiThread
{
    class ConfigureAwait
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Main1:" + Thread.CurrentThread.ManagedThreadId);
            _ = ConfigureAwaitTest();
            Console.WriteLine("Main2:" + Thread.CurrentThread.ManagedThreadId);
            Thread.Sleep(3000);
        }

        public async static Task ConfigureAwaitTest()
        {
            Console.WriteLine("ConfigureAwaitTest_Before thread is " + Thread.CurrentThread.ManagedThreadId);
            await Task.Run(() =>
            {
                Thread.Sleep(1000);
                Console.WriteLine("task thread is " + Thread.CurrentThread.ManagedThreadId);
            });
            Console.WriteLine("ConfigureAwaitTest_After thread is " + Thread.CurrentThread.ManagedThreadId);
        }
    }
}
```

> Main1:1
> ConfigureAwaitTest\_Before thread is 1
> Main2:1
> task thread is 4
> ConfigureAwaitTest\_After thread is 4

设置ConfigureAwait(true):

```c#
await Task.Run(() =>
{
	//...
}).ConfigureAwait(true);
```

运行结果与默认设置相同。同时，设置ConfigureAwait(false)时，运行结果并没有变化；总之，在非UI场景下，ConfigureAwait(bool)并无明显变化。

##### UI场景

**设置ConfigureAwait(true) 或者 默认情况下,** 运行结果：

> Main1: current\_thread: 1
> ConfigureAwaitTest\_Before thread is 1
> Main2:   current\_thread: 1
> task thread is 3
> ConfigureAwaitTest\_After thread is 1

最终发现，在异步任务中，await执行之后，将剩下的工作交由主线程执行而非Task线程！

**设置ConfigureAwait(false),** 运行结果：

> Main1: current\_thread: 1
> ConfigureAwaitTest\_Before thread is 1
> Main2:   current\_thread: 1
> task thread is 3
> ConfigureAwaitTest\_After thread is 3

主要是await之后的方法仍旧由task线程执行！

##### 引申1

无法在异步线程中执行UI操作，因此如下代码：

```c#
await Task.Run(() =>
{
	Thread.Sleep(1000);
    Console.WriteLine("task thread is " + Thread.CurrentThread.ManagedThreadId);
    label.Text = "new_value " + 1024;
}).ConfigureAwait(true);
```

会出现错误！可以修改为：

```c#
await Task.Run(() =>
{
	Thread.Sleep(1000);
    Console.WriteLine("task thread is " + Thread.CurrentThread.ManagedThreadId);
}).ConfigureAwait(true);
label.Text = "new_value " + 1024;  // 设置为 true之后， 该行代码交由 调用方 执行，因此可以修改UI
```

如果是UI上下文，有大量的async方法在UI上下文恢复，会引起性能的问题。

在编写 async 代码时特别注意上下文。通常一个 async 方法要么需要上下文（处理 UI 元素或 ASP.NET 请求 / 响应），要么需要摆脱上下文（执行后台指令）。如果一个 asnync 方法的一部分需要上下文、一部分不需要上下文，则可考虑把它拆分为两个（或更多）async 方法。这种做法有利于更好地将代码组织成不同层次。

##### 引申2

```c#
private void button1_Click(object sender, EventArgs e)
{
    Task task = TestConfigureAwait();
    //等待task执行完成
    task.wait();
}
public async Task TestConfigureAwait()
{
    await Task.Run(() =>
    {
        // ...
    }).ConfigureAwait(true);
}
```

执行TestConfigureAwait到await时候，线程阻塞，返回到调用方 task.Wait();

此时，ConfigureAwait(true) 设置将返回到调用方，而此时调用方阻塞；因此会造成死锁！ 修改 ： ConfigureAwait(false);