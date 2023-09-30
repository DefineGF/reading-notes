#### 基本框架

##### Selector

```java
Selector selector = Selector.open();
```

- isOpen()：是否打开；
- close()：进入关闭状态；且注册该Selector上的所有SelectorKey实例无效；通道本身不会关闭；

含有三种类型 SelectionKey：

- all-keys：当前所有向Selector注册的SelectionKey集合，Selector 的 keys() 方法返回集合；
- selected-keys：已被Selector捕获的SelectionKey的集合，Selector 的 selectedKeys() 返回集合；
- cancelleded-keys：已经被取消的SelectionKey集合



##### SelectionKey

- SelectionKey.OP_CONNECT       //某channel成功连接到另一个服务器
- SelectionKey.OP_ACCEPT          //ServerSocketChannel准备好接受新的连接
- SelectionKey.OP_READ              //有数据可读的通道
- SelectionKey.OP_WRITE             //有可写数据的通道

```java
//interest集合 —— 感兴趣的事件集合，可以通过SelectionKey读写interest集合
boolean isInterestedInAccept = 
			(interestSet & Selection.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
			
boolean isInterestedInConnect = interestSet & SelectioKey.OP_CONNECT;
boolean isInterestedInRead = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite = interestSet & SelectionKey.OP_WRITE;

//ready集合 —— 是通道已经准备就绪的操作的集合，在一个选择后，你会是首先访问这个ready set
int readySet = selectionKey.readyOps();
selectionKey.isAcceptable(); //等价于selectionKey.readyOps() & SelectionKey.OP_ACCEPT
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();

//获取关联的Selector和Channel
Channel channel =selectionKey.channel(); 
Selector selector=selectionKey.selector();
```



绑定对象：

- 在注册的时候直接绑定

  ```java
  SelectionKey key=channel.register(selector,SelectionKey.OP_READ,theObject);
  ```

- 在绑定完成之后附加:

  ```java
  selectionKey.attach(theObject);//绑定
  ```

取出对象：

```java
selectionKey.attachment();
```

取消对象：

```java
selectionKey.attach(null)；//一定要认为回收，垃圾回收器不会回收而导致内存泄漏
```



##### 过程

- 当 register() 方法执行时，新建一个 SelectionKey，并加入 Selector 的 all-keys 集合中；

- 如果关闭了与SelectionKey对象关联的Channel对象，或者调用了SelectionKey对象的cancel方法，

  这个SelectionKey对象就会被加入到cancelled-keys集合中，表示这个SelectionKey对象已经被取消



注册通道：

```java
ServerSocketChannel serverChannel = ServerSocketChannel.open();

serverChannel.socket().bind(new InetSocketAddress(6789));
serverChannel.configureBlocking(false);//非阻塞
// Selector就会监控这些事件是否发生
SelectionKey key = channel.register(selector,SelectionKey.OP_READ);
```

