### 用法汇总

基本使用

```java
RequestBody requestBody=new FormBody.Builder().add("key","value").build();
```



#### 执行方式

##### 同步方式

```java
Response response=client.newCall(request).execute();
```



##### 异步方式

响应：

```java
Response response = client.newCall(request).enqueue(new Callback(){
        @Override
        public void onFailure(Request request, IOException e) { // 失败回调
         
        } 
        @Override 
        public void onResponse(Response response) throws IOException {
         //成功 注意：这里是后台线程！
        }
        
    }    
);
```

请求：

```java
FormBody formBody = new FormBody.Builder()
                              .add("Uname",name)
                              .add("Upwd",password)
                              .build();

Request request = new Request.Builder()
                           .url(url)
                           .post(fromBody)
                           .build();
```

请求对象为键值对：FormBody

请求对象为对象：RequestBody

##### 发送Json

```java
private static final MediaType JSON = 
							MediaType.parse("application/json;charset=UTF-8");
Gson gson=new Gson();
String goodsJson=gson.toJson(goods);

RequestBody requestBody = RequestBody.create(JSON,goodsJson);
Request request=new Request.Builder()
        .post(requestBody)
        .url("http://www.baidu.com")
        .build();
OkHttpClient client = new OkHttpClient();
try {
    return client.newCall(request).execute();// 返回类型Response
} catch (IOException e) {
    e.printStackTrace();
}
return null;
```





##### 封装

```java
public class NetUtil {
    public static void connByRequest(Request request, Callback callBack){
        OkHttpClient client=new OkHttpClient();
        Call call = client.newCall(request);
        call.enqueue(callBack);
    }
    
    public static void connByGET(String url,Callback callback){
        Request request=new Request.Builder()
                .url(url)
                .method("GET",null)
                .build();
        connByRequest(request,callback);
    }
    public static void connByPost(String url, RequestBody body, Callback callback){
        Request request=new Request.Builder()
                .url(url)
                .post(body)
                .build();
        connByRequest(request,callback);
    }
}
```



#### 源码解析

- OkHttpClient 实现 Call.Factory，负责为 Request 创建 Call；

- RealCall 为具体的 Call 实现，其 enqueue() 异步接口通过 Dispatcher 利用 ExecutorService 实现，而最终进行网络请求时和同步 execute() 接口一致，都是通过 getResponseWithInterceptorChain() 函数实现；

- getResponseWithInterceptorChain() 中利用 Interceptor 链条，分层实现缓存、透明压缩、网络 IO 等功能；

##### OkHttp.Builder

```java
final Dispatcher dispatcher; //分发器 
final Proxy proxy; //代理
final List protocols; //协议 
final List connectionSpecs; //传输层版本和连接协议 
final List interceptors; //拦截器 
final List networkInterceptors; //网络拦截器 
final ProxySelector proxySelector; //代理选择
final CookieJar cookieJar; //cookie 
final Cache cache; //缓存 
final InternalCache internalCache; //内部缓存 
final SocketFactory socketFactory; //socket 工厂 
final SSLSocketFactory sslSocketFactory; //安全套接层socket 工厂，用于HTTPS
final CertificateChainCleaner certificateChainCleaner; // 验证确认响应证书 适用 HTTPS 请求连接的主机名。 
final HostnameVerifier hostnameVerifier; // 主机名字确认 
final CertificatePinner certificatePinner; // 证书链 
final Authenticator proxyAuthenticator; //代理身份验证 
final Authenticator authenticator; // 本地身份验证 
final ConnectionPool connectionPool; //连接池,复用连接 
final Dns dns; //域名 
final boolean followSslRedirects; //安全套接层重定向 
final boolean followRedirects; //本地重定向 
final boolean retryOnConnectionFailure; //重试连接失败 
final int connectTimeout; //连接超时 
final int readTimeout; //read 超时 
final int writeTimeout; //write 超时

// 默认设置
public Builder() {
  dispatcher = new Dispatcher();
  protocols = DEFAULT_PROTOCOLS;
  connectionSpecs = DEFAULT_CONNECTION_SPECS;
  eventListenerFactory = EventListener.factory(EventListener.NONE);
  proxySelector = ProxySelector.getDefault();
  cookieJar = CookieJar.NO_COOKIES;
  socketFactory = SocketFactory.getDefault();
  hostnameVerifier = OkHostnameVerifier.INSTANCE;
  certificatePinner = CertificatePinner.DEFAULT;
  proxyAuthenticator = Authenticator.NONE;
  authenticator = Authenticator.NONE;
  connectionPool = new ConnectionPool();
  dns = Dns.SYSTEM;
  followSslRedirects = true;
  followRedirects = true;
  retryOnConnectionFailure = true;
  connectTimeout = 10_000;
  readTimeout = 10_000;
  writeTimeout = 10_000;
  pingInterval = 0;
}
```

