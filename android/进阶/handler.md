### handler

在多线程的应用场景中，将**工作线程**中需更新UI的操作信息 传递到 **UI主线程**，从而实现工作线程对UI的更新处理，最终实现异步消息的处理



#### 工作过程

##### 准备

主线程创建以下对象：

- Looper：创建完 MessageQueue之后，Looper 自动进入消息循环；
- MessageQueue
- Handler

Handler 主动绑定了主线程的 Looper、MessageQueue



##### 消息入队

工作线程通过 Handler 将Message 发送到消息队列 MessageQueue



##### 消息循环

- 消息出队：looper 循环取出队列中的消息；
- 消息分发：looper将取出的消息分发给对应的处理者；

若队列为空，则阻塞。



##### 消息处理

- handler 接收 looper 发送来的 message
- handler 根据message 进行ui操作



#### 创建 Looper

在 Android 中，线程不是自带 Looper 的，只有主线程（即主 UI 线程）会自动创建一个 Looper 对象并关联到当前线程。

主线程的 Looper 负责处理消息队列中的消息，并将它们分发给对应的 Handler 进行处理。这样可以确保在主线程上执行 UI 相关的操作，因为 UI 操作必须在主线程上执行，以避免多线程并发引起的问题。

而对于其他线程（非主线程），默认是没有 Looper 的。如果你想在其他线程中使用 Handler，需要先调用 `Looper.prepare()` 创建一个 Looper，并调用 `Looper.loop()` 启动消息循环。

当然也可以使用以下方法创建：

```java
HandlerThread thread = new HandlerThread("ServiceHandlerThread", Process.THREAD_PRIORITY_BACKGROUND);
Looper looper = thread.getLooper();// 获取工作线程的Looper
```



