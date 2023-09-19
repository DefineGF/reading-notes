### Service

虽然service在后台运行，但是依旧实在主线程中进行所有操作；

除非特殊设置，任何能阻塞主线程的操作（比如网络请求，音乐播放等）都应该开辟新的线程来操作；（android提供intentService来完成异步操作）

##### 启动 & 关闭

- 启动

  - context.startService()

    一般情况下，如果只需要一次单独操作，不需要将结果返回时，使用**前者**；

    通过stopService或者service自身调用stopSelf来销毁；

    >context.startService() --> onCreate() --> onStart()  
    >
    >	service running...
    >
    >context.stopService() --> onDestroy()

  - context.bindService()

    后者通过组件绑定服务发送请求并获取结果时使用；

    通过组件解绑或者销毁组件来终止service，一个serveice可由多个组件绑定；

    > context.bindService() --> onCreate() --> onBind() 
    >
    > 	 service running...
    >
    > unBind() --> onDestory()

##### 生命周期

 <img src="F:\Typora\Nodes\Android\Base\Image-1615984012921.png" alt="Image" style="zoom:67%;" />

无论通过哪种方式，都会调同onCreate 和 onDestroy；

以音乐播放器为例，需要在onCreate创建播放线程，在onDestroy进行资源回收；



##### context.startService():

context.stopService() 或者service中的 stopSelf来停止;

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



##### context.bindService()：

销毁：bindService启动的service生命周期和与其绑定的client息息相关；

当client销毁时候，service自动解除绑定并销毁；当然也可以通过context.unbindService()解除绑定；



##### service 与 broadcast

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
        HandlerThread thread = new HandlerThread(
            "ServiceHandlerThread", Process.THREAD_PRIORITY_BACKGROUND);
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

服务启动Activity：

```java
private ImageView mImageView;
private BroadcastReceiver mReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 显示图片
        Bitmap bitmap = intent.getParcelableExtra("bitmap");
        mImageView.setImageBitmap(bitmap);
    }
};


/**
 * 启动下载
 */
private void startDownload(){
	Intent intent = new Intent(this, DownloadService.class);
    intent.putExtra(DownloadService.IMAGE, "http://www.8kmm.com/UploadFiles/2012/8/201208140920132659.jpg");
    intent.setAction(DownloadService.ACTION_DOWNLOAD);
    startService(intent);
}
```



#### 下载文件

```java
public interface DownloadListener {
    void onProgress(int progress);
    void onSuccess();
    void onFailed();
    void onPaused();
    void onCanceled();
}
```



下载任务：

```java
public class DownloadTask extends AsyncTask<String, Integer, Integer> {
    public static final int TYPE_SUCCESS = 0;
    public static final int TYPE_FAILED = 1;
    public static final int TYPE_PAUSED = 2;
    public static final int TYPE_CANCELED = 3;
    private DownloadListener listener;
    private boolean isCanceled = false;
    private boolean isPaused = false;
    private int lastProgress;
    
    public DownloadTask(DownloadListener listener) {
        this.listener = listener;
    }
    @Override
    protected Integer doInBackground(String... params) {
        InputStream is = null;
        RandomAccessFile savedFile = null;
        File file = null;
        try {
            long downloadedLength = 0; // 记录已下载的文件长度
            String downloadUrl = params[0];
            String fileName = downloadUrl.substring(downloadUrl.lastIndexOf("/"));
            String directory = Environment.getExternalStoragePublicDirectory
                (Environment.DIRECTORY_DOWNLOADS).getPath();
            file = new File(directory + fileName);
            if (file.exists()) {
                downloadedLength = file.length();
            }
            long contentLength = getContentLength(downloadUrl);
            if (contentLength == 0) {
                return TYPE_FAILED;
            } else if (contentLength == downloadedLength) {
                // 已下载字节和文件总字节相等，说明已经下载完成了
                return TYPE_SUCCESS;
            }
            OkHttpClient client = new OkHttpClient();
            Request request = new Request.Builder()
                    // 断点下载，指定从哪个字节开始下载
                    .addHeader("RANGE", "bytes=" + downloadedLength + "-")
                    .url(downloadUrl)
                    .build();
            Response response = client.newCall(request).execute();
            if (response ! = null) {
                is = response.body().byteStream();
                savedFile = new RandomAccessFile(file, "rw");
                savedFile.seek(downloadedLength); // 跳过已下载的字节
                byte[] b = new byte[1024];
                int total = 0;
                int len;
                while ((len = is.read(b)) ! = -1) {
                    if (isCanceled) {
                        return TYPE_CANCELED;
                    } else if(isPaused) {
                        return TYPE_PAUSED;
                    } else {
                        total += len;
                        savedFile.write(b, 0, len);
                        // 计算已下载的百分比
                        int progress = (int) ((total + downloadedLength) * 100 /
                            contentLength);
                        publishProgress(progress);
                    }
                }
                response.body().close();
                return TYPE_SUCCESS;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (is ! = null) {
                    is.close();
                }
                if (savedFile ! = null) {
                    savedFile.close();
                }
                if (isCanceled && file ! = null) {
                    file.delete();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return TYPE_FAILED;
    }
    @Override
    protected void onProgressUpdate(Integer... values) {
        int progress = values[0];
        if (progress > lastProgress) {
            listener.onProgress(progress);
            lastProgress = progress;
        }
    }
    @Override
    protected void onPostExecute(Integer status) {
        switch (status) {
            case TYPE_SUCCESS:
                listener.onSuccess();
                break;
            case TYPE_FAILED:
                listener.onFailed();
                break;
            case TYPE_PAUSED:
                listener.onPaused();
                break;
            case TYPE_CANCELED:
                listener.onCanceled();
                break;
                    default:
                        break;
                }
            }
            public void pauseDownload() {
                isPaused = true;
            }
            public void cancelDownload() {
                isCanceled = true;
            }
            private long getContentLength(String downloadUrl) throws IOException {
                OkHttpClient client = new OkHttpClient();
                Request request = new Request.Builder()
                        .url(downloadUrl)
                        .build();
                Response response = client.newCall(request).execute();
                if (response ! = null && response.isSuccessful()) {
                    long contentLength = response.body().contentLength();
                    response.body().close();
                    return contentLength;
                }
                return 0;
            }
        }
```

