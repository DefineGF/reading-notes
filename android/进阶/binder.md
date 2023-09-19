### Binder实现原理



#### Binder 例子

##### service

```java
public class MyService extends Service {
    private final IBinder binder = new MyBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }

    public class MyBinder extends Binder {
        public int add(int a, int b) {
            return a + b;
        }
    }
}
```

##### Activity

```java
public class MyActivity extends Activity {
    private MyService.MyBinder binder;
    private boolean isServiceBound = false;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            binder = (MyService.MyBinder) service;
            isServiceBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            isServiceBound = false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        Intent intent = new Intent(this, MyService.class);
        bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        if (isServiceBound) {
            unbindService(serviceConnection);
            isServiceBound = false;
        }
    }

    private void performOperation() {
        if (isServiceBound) {
            int result = binder.add(5, 3);
            Toast.makeText(this, "Result: " + result, Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(this, "Service not bound", Toast.LENGTH_SHORT).show();
        }
    }
}
```

#### 工作原理

Binder 的工作过程的详细步骤：

1. 客户端调用代理对象的方法：
   客户端通过 Binder 代理对象调用方法，并传递参数。代理对象实际上是客户端进程中的一个本地对象，它封装了对远程 Binder 对象的访问。
2. 代理对象将请求发送给 **Binder 驱动程序**：
   代理对象将请求打包，包括方法名和参数，并将请求发送给客户端进程的 Binder 驱动程序。这个请求被称为 Binder Transaction。
3. Binder 驱动程序将请求发送给服务端的 Binder 线程：
   Binder 驱动程序接收到客户端的请求后，将其发送给服务端进程的 Binder 线程。这个过程涉及进程间的内核间通信。
4. 服务端的 Binder 线程将请求传递给存根对象：
   服务端的 Binder 线程接收到来自客户端的请求后，将其传递给服务端进程中的存根对象。存根对象实际上是服务端进程中的一个本地对象，它实现了与客户端相同的接口。
5. 存根对象解析请求并调用实际方法：
   存根对象解析请求中的方法名和参数，并调用实际的方法。这些方法执行的是服务端进程中的本地逻辑，可能包括数据处理、计算等操作。
6. 存根对象将方法执行结果返回给 Binder 线程：
   存根对象执行方法后，将方法执行的结果打包，并将结果返回给服务端进程的 Binder 线程。
7. Binder 线程将结果传递给客户端的 Binder 线程：
   服务端的 Binder 线程接收到方法执行结果后，将结果传递给客户端进程的 Binder 线程。这个过程同样涉及进程间的内核间通信。
8. 代理对象将结果返回给客户端：
   客户端的 Binder 线程接收到方法执行结果后，将结果传递给客户端进程中的代理对象。代理对象解包结果，并返回给客户端的调用方。

##### binder底层驱动程序

Binder 机制的底层驱动程序是 Binder 驱动模块（binder.ko）。这个驱动模块是 Android 系统的一部分，它在内核层面实现了 Binder IPC 机制。负责管理进程间通信所需的内核数据结构，并提供了对这些数据结构的操作接口。

1. Binder 节点（Binder Node）：
   Binder 节点是用于存储进程中的 Binder 对象的数据结构。每个进程都有一个与之对应的 Binder 节点。Binder 节点通过链表的形式连接在一起，形成进程间通信的网络。
2. Binder 线程（Binder Thread）：
   Binder 线程是负责处理进程间通信请求和响应的线程。每个进程都有一个 Binder 线程，它负责接收和发送 Binder Transaction。
3. Binder Transaction：
   Binder Transaction 是进程间通信的基本单元。它是一种用于打包和传输数据的数据结构，包括方法调用信息、参数和返回值等。Binder Transaction 通过 Binder 节点和 Binder 线程进行传输。
4. Binder 驱动接口：
   Binder 驱动模块提供了一组接口，供上层用户空间程序（如 Java 层的 Binder 框架）与驱动程序进行交互。这些接口包括创建 Binder 节点、发送和接收 Binder Transaction 等。