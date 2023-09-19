### 常见内存泄露

#### 单例模式

单例的静态特性导致它的生命周期和整个应用的生命周期一样长，如果有对象已经不再使用了，但又却被单例持有引用，那么就会导致这个对象就没办法被回收，从而导致内存泄漏。

例子如下：

```java
// 使用了单例模式
public class AppManagerXiangxue {
    private static AppManagerXiangxue instance;
    private Context context;
    private AppManagerXiangxue(Context context) {
        this.context = context;
    }
    public static AppManagerXiangxue getInstance(Context context) {
        if (instance != null) {
            instance = new AppManagerXiangxue(context);
        }
        return instance;
    }
}
```

上面的代码我们可以看出，在创建单例对象的时候，引入了一个Context上下文对象，如果我们把Activity注入进来，会导致这个Activity一直被单例对象持有引用，当这个Activity销毁的时候，对象也是没有办法被回收的。

**解决方案**:  在这里我们只需要让这个上下文对象指向应用的上下文即可（this.context=context.getApplicationContext()），因为应用的上下文对象的生命周期和整个应用一样长。



#### 非静态内部类创建静态实例

例子如下：

```java
public class MainActivity extends AppCompatActivity {
    private static TestResource mResource = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mResource == null){
            mResource = new TestResource();
        }
        //...
    }
   
    class TestResource {
    //...
    }
}
```

- 由于静态成员变量是与类本身相关联的，它的生命周期通常比较长;
- 非静态内部类的实例通常与外部类实例相关联，如果它被持有并且不会被及时释放，就会造成外部类实例无法被垃圾回收，从而导致内存泄漏。

常用解决手段：

1. 将内部类声明为静态：将内部类声明为静态内部类，这样它就不会隐式地持有对外部类实例的引用。
2. 显式地释放引用：在不再需要内部类实例时，显式地将其引用置为 `null`，以便让垃圾回收器回收内存。
3. 使用弱引用：可以考虑使用弱引用（WeakReference）来持有对内部类实例的引用，这样当没有其他强引用指向内部类实例时，它就可以被垃圾回收



#### handler

首先我们知道程序启动时在主线程中会创建一个Looper对象，这个Looper里维护着一个MessageQueue消息队列，这个消息队列里会按时间顺序存放着Message；

Handler是通过内部类来创建的，内部类会持有外部类的引用，也就是Handler持有Activity的引用，而消息队列中的消息target是指向Handler的；

也就等同消息持有Handler的引用，也就是说当消息队列中的消息如果还没有处理完，这些未处理的消息（也可以理解成延迟操作）是持有Activity的引用的；

(message -> handler -> activity)；

此时如果关闭Activity，是没办法回收的，从而就会导致内存泄露。

解决方案：

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

     new Thread(new Runnable() {
        @Override
        public void run() {
            myHandler.sendMessage(Message.obtain());
        }
    }).start();
}

@Override
protected void onDestroy() {
    super.onDestroy();
    //移除对应的Runnable或者是Message
    //mHandler.removeCallbacks(runnable);
    //mHandler.removeMessages(what);
    mHandler.removeCallbacksAndMessages(null);
}

MyHandler myHandler = new MyHandler(this);

private static class MyHandler extends Handler {
    private WeakReference<Activity> mActivity; // 弱引用

    public MyHandler(Activity activity) {
        mActivity = new WeakReference<Activity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
        if (mActivity.get() == null) {
            return;
        }
         //to do something..

    }
};
```

