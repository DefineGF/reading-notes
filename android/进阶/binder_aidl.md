### AIDL

AIDL 全称是 Android Interface Definition Language(Android接口描述语言)是一种接口描述语言。编译器可以通过aidl文件生成一段代码，生成的代码封装了binder，可以当成是binder的延伸.



#### 使用过程

##### 服务端

1. 创建aidl接口；
2. 实现接口，并向客户端开放接口；
3. 创建服务，返回binder

##### 客户端

1. 绑定服务；
2. 实现ServiceConnection绑定监听；
3. 在绑定成功的回调中，将IBinder转换成AIDL的接口代理对象



#### 绑定过程

- 客户端先调用bindService方法，发起绑定服务的请求，通过ServiceManager，拿到ActivityManagerService，也就是AMS，然后通过AMS向服务端发起bindService的请求；
- 服务端接收到绑定请求，以Handler消息机制的方式，发送一个绑定服务的Message；
- 在ActivityThread中处理这个绑定请求，调用onBind函数，并返回对应的IBinder对象。这个返回IBinder对象的操作，基本就和绑定过程的通过ServiceManager、AMS类似了。