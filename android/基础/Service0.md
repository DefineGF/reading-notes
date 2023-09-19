### Service

#### 启动

##### context.startService()

- 一般情况下，如果只需要一次单独操作，不需要将结果返回时，使用该方法；
- 通过stopService或者service自身调用stopSelf来销毁；

当系统内存不足时，会销毁当前service，等到内存充足时候再根据**onStartCommand返回值**重新创建：

- START_NOT_STICKY：

  service被系统销毁后，不会自动重新创建；  

  适用于service被杀死，且不会立刻重新创建也不会有太大影响的service；

- START_STICKY：

  service销毁后，会保持started状态，但不保留传入的intent信息，然后重建service，

  接着**调用onStartCommand**（其中intent为null）

  适用于：service任何时候结束都没问题，且不需要intent信息

- START_REDELIVER_INTENT：

  使用保存的intent重新创建service；



生命周期：

1. startService()
2. onCreate()
3. onStart()
4. stopService()
5. onDestory()



##### context.bindService()

bindService启动的service生命周期和与其绑定的client息息相关：

- 当client销毁时候，service自动解除绑定并销毁；
- 也可以通过context.unbindService()解除绑定；

生命周期：

1. startService()
2. onCreate()
3. onBind()
4. unBind()
5. onDestory()



#### 实例

##### 图片下载并显示

启动下载：

```java
private void startDownload(){
	Intent intent = new Intent(this, DownloadService.class);
    intent.putExtra(DownloadService.IMAGE, "http://www.8kmm.com/UploadFiles/2012/8/201208140920132659.jpg");
    intent.setAction(DownloadService.ACTION_DOWNLOAD);
    startService(intent);
}
```

下载服务：

```java
public class DownloadService extends Service {
    public static final String RECEIVER_ACTION = "com.zhouwei.simpleservice";
    public static final String ACTION_START_SERVICER = "com.zhouwei.startservice";
    public static final String ACTION_DOWNLOAD = "com.zhouwei.startdownload";
    
    public static final String IMAGE = "iamge_url";
    
    private Looper mServiceLooper;
    private ServiceHandler mServiceHandler;

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    @Override
    public void onCreate() {
        // 开启一个工作线程做耗时工作
        HandlerThread thread = new HandlerThread("ServiceHandlerThread", Process.THREAD_PRIORITY_BACKGROUND);
        thread.start();

        mServiceLooper = thread.getLooper();// 获取工作线程的Looper
        mServiceHandler = new ServiceHandler(mServiceLooper); // 创建工作线程的Handler
    }


    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.i(TAG,"call onStartCommand...");
        if(intent.getAction().equals(ACTION_DOWNLOAD)){
            handleCommand(intent);
        }else if(intent.getAction().equals(ACTION_START_SERVICER)){
            //do nothing
        }
        return START_STICKY;
    }

    private void handleCommand(Intent intent){
        String url = intent.getStringExtra(IMAGE);
        // 发送消息下载
        Message message = mServiceHandler.obtainMessage();
        message.obj = url;
        mServiceHandler.sendMessage(message);
    }
    
    // handler
    private final class ServiceHandler extends Handler {
        
        public ServiceHandler(Looper looper){
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            // 工作线程做耗时下载
            String url = (String) msg.obj;
            Bitmap bitmap = null;
            try {
                bitmap = Picasso.with(getApplicationContext()).load(url).get();
                Intent intent = new Intent();
                intent.putExtra("bitmap", bitmap);
                intent.setAction(RECEIVER_ACTION);
                
                // 通知显示
                LocalBroadcastManager
                    .getInstance(getApplicationContext())
                    .sendBroadcast(intent);
            } catch (IOException e) {
                e.printStackTrace();
            }
            stopSelf(); //工作完成之后，停止服务
        }
    }
}
```

