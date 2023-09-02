### AutoResetEvent

#### 构造

```c#
private static AutoResetEvent event_1 = new AutoResetEvent(true);
private static AutoResetEvent event_2 = new AutoResetEvent(false);
```

#### 操作

##### WaitOne()

如果当前AutoResetEvent的状态为true，则当前不会阻塞。同时将状态设置为false，意味着下次执行到此时会阻塞。
如果当前状态为false，则线程执行到此会阻塞；

##### Set()

将当前状态设置为true，如果有被event.WaitOne()阻塞的，则唤醒。

#### 实例

```c#
private static AutoResetEvent event_1 = new AutoResetEvent(true);
private static AutoResetEvent event_2 = new AutoResetEvent(false);
for (int i = 1; i < 4; i++)
{
    Thread t = new Thread(ThreadProc);
    t.Name = "Thread_" + i;
    t.Start();
}

static void ThreadProc()
{
    string name = Thread.CurrentThread.Name;

    Console.WriteLine("{0} waits on AutoResetEvent #1.", name);
    event_1.WaitOne();
    Console.WriteLine("{0} is released from AutoResetEvent #1.", name);

    Console.WriteLine("{0} waits on AutoResetEvent #2.", name);
    event_2.WaitOne();
    Console.WriteLine("{0} is released from AutoResetEvent #2.", name);

    Console.WriteLine("{0} ends.", name);
}
```

首先，for循环中会创建三个线程，加入第一个线程为thread\_1，则其运行状态为：

```shell
Thread_1 waits on AutoResetEvent #1.
Thread_1 is released from AutoResetEvent #1.
Thread_1 waits on AutoResetEvent #2.
```

之所以跨过event\_1.WaitOne(),则是由于其创建状态为true；

然后是：

    Thread_3 waits on AutoResetEvent #1.
    Thread_2 waits on AutoResetEvent #1.

接着，如果在代码中加入两个循环体：

```c#
for (int i = 0; i < 2; i++)
{
    Console.WriteLine("Press Enter to release another thread.");
    Console.ReadLine();
    event_1.Set();
    Thread.Sleep(250);
}

Console.WriteLine("\r\nAll threads are now waiting on AutoResetEvent #2.");
for (int i = 0; i < 3; i++)
{
    Console.WriteLine("Press Enter to release a thread.");
    Console.ReadLine();
    event_2.Set();
    Thread.Sleep(250);
}
```

回车1：执行event\_1.Set(), 被event\_1.WaitOne() 阻塞的thread\_3继续执行

> Thread\_3 is released from AutoResetEvent #1.
> Thread\_3 waits on AutoResetEvent #2.

回车2：再次执行event\_1.Set(), 被event\_1.WaitOne() 阻塞的thread\_2继续执行：

> Thread\_2 is released from AutoResetEvent #1.
> Thread\_2 waits on AutoResetEvent #2.

这个时候：
All threads are now waiting on AutoResetEvent #2.

紧接着，连续按三次回车，则三个线程陆续执行完毕。
