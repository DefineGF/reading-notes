#### 缓冲区 Buffer

##### 构造

```java
CharBuffer charBuffer = CharBuffer.allocate(100); 
// 关联数组
char[] arr = new char[100];
CharBuffer charBuf = CharBuffer.wrap(arr);
```



##### 属性

0 <= mark <= position <= limit <= capacity；

- capacity：容量，一旦创建便不会更改；
- limit：缓冲区现存元素的计数；
- position：下一个要被读或写的元素索引。可通过get() 或者 put() 函数更改；
- mark：备忘位置。
  - mark()：mark = position；
  - reset()：position = mark; 

##### API

```java
public abstract class Buffer { 
 public final int capacity( ) 
 public final int position( ) 
 public final Buffer position (int newPosition)
 public final int limit( ) 
 public final Buffer limit (int newLimit) 
 public final Buffer mark() 
 public final Buffer reset() 
 public final Buffer clear() 
 public final Buffer flip();   // == buf.limit(buf.position()).positon(0); 
 public final Buffer rewind( ) 
 public final int remaining();  // 剩余元素数
 public final boolean hasRemaining( ) 
 public abstract boolean isReadOnly( ); 
}
```

compact()：

1. 将position和limit之间的数据复制到buffer的开始位置（index = 0），
2. position = limit - poition;
3. limit = capacity;

##### 读写数据，四个步骤：

- 写入buffer

- 调用flip --> 从写模式转换为读模式

- 从buffer读取数据

- 调用buffer.clear()

  ```java
  public final Buffer clear() {
      position = 0;
      limit = capacity;
      mark = -1;
      return this;
  }
  ```

完整实例：

```java
public class BufferTest {
    private static int index = 0;
    private final static String[] strings = {
        "十里寒塘路",
        "烟花一半醒",
    };

    public static void main(String[] args) {
        CharBuffer buffer = CharBuffer.allocate(100);
        while (fillBuffer(buffer)) {
            buffer.flip();
            drainBuffer(buffer);
            buffer.clear();
        }
    }

	// 写
    private static boolean fillBuffer(CharBuffer buffer) {
        if (index >= strings.length) {
            return false;
        }
        String str = strings[index++];
        for (int i = 0; i < str.length(); i++) {
            buffer.put(str.charAt(i));
        }
        return true;
    }
	
	// 读
    private static void drainBuffer(CharBuffer buffer) {
        while (buffer.hasRemaining()) {
            System.out.println("get = " + buffer.get());
        }
    }
}
```





##### 文件复制实例

```
 private static void copyData(ReadableByteChannel src, WritableByteChannel dest) 
 														       throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(20 * 1024);
        while (src.read(buffer) != -1) { // from channel to buffer
            buffer.flip();
            while (buffer.hasRemaining()) {
                dest.write(buffer);
            }
            buffer.clear();
        }
    }
```



### 通道 - channel

IO可广义分为两类：File IO和 Stream IO，因此对应两种通道，分别是文件通道和socket通道。FileChannel、SocketChannel、ServerSocketChannel和DatagramChannel。

##### 相关定义

```java
public interface Channel {
	public boolean isOpen();
	public void close() throws IOException;
}
public interface ReadableByteChannel extends Channel {
	public int read (ByteBuffer dst) throws IOException;
}
public interface WritableByteChannel extends Channel {
	public int write (ByteBuffer src) throws IOException;
}
public interface ByteChannel extends ReadableByteChannel, WritableByteChannel {}
```

注意：

一个文件可以在不同时候以不同权限打开。从FileInputStream 对象 getChannel() 方法获取的 FileChannel对象是可读的；但是 FileChannel实现了ByteChannel接口，后者是双向的。因此在这个Channel调用write() 将抛出NonWritableChannelException 异常。 

##### 通道数据复制

```java
public class ChannelCopy {

    public static void main(String[] args) throws IOException  {
        ReadableByteChannel source = Channels.newChannel(System.in);
        WritableByteChannel dest = Channels.newChannel(System.out);
        // channelCopy1(source, dest);
        channelCopy2(source, dest);
        source.close();
        dest.close();
    }
    private static void channelCopy1(ReadableByteChannel src, WritableByteChannel dest)
                                                                     throws IOException {
        ByteBuffer buf = ByteBuffer.allocate(16 * 1024);
        while (src.read(buf) != -1) {
            buf.flip();
            dest.write(buf);
            buf.compact();
        }
        buf.flip();
        while (buf.hasRemaining()) {
            dest.write(buf);
        }
    }

    private static void channelCopy2(ReadableByteChannel src, WritableByteChannel dest)
                                                                      throws  IOException{
        ByteBuffer buf = ByteBuffer.allocate(16 * 1024);
        while (src.read(buf) != -1) {
            buf.flip();
            while (buf.hasRemaining()) {
                dest.write(buf);
            }
            buf.clear();
        }
    }
}
```



##### scatter/gather

```java
// 从通道读取的数据会按顺序被散布到多个缓冲区
public interface ScatteringByteChannel extends ReadableByteChannel {
	public long read(ByteBuffer[] dsts) throws IOException;
	public long read(ByteBuffer[] dsts, int offset, int length) throws IOException;
}
// 数据从几个缓冲区按顺序抽取并沿着通道发送的
public interface GatheringByteChannel extends WritableByteChannel {
	public long write(ByteBuffer[] srcs) throws IOException;
	public long write(ByteBuffer[] srcs, int offset, int length);
}
```

使用

```java
// scatter 使用
ByteBuffer header = ByteBuffer.allocateDirect(10);
ByteBuffer body = ByteBuffer.allocateDirect(80);
int bytesRead = channel.Read({header, body});	// 假设channel连接到一个有48字节等待读取的socket上
if (header.getShort(0) == TYPE_FILE) {
    body.flip();
    fileChannel.write(body);
}

// gather 使用
body.clear();
body.put("data".getBytes()).flip();
header.clear();
header.putShort(TYPE_FILE).putLong(body.limit()).flip();
long bytesWritten = channel.write({header, body});
```



#### 文件通道

##### 定义

```java
public abstract class FileChannel extends AbstractChannel 
		implements ByteChannel, GatheringByteChannel, ScatteringByteChannel {
    // partial api 
	public abstract long position(); // position决定文件中哪个位置的数据将要被读或者写
	public abstract void position(long newPosition); // 将通道的position设置成指定的position
    public abstract void truncate(long size); // set length
    
    // 告诉通道强制将全部待定的修改都应用到磁盘文件上
    // metaData: 元数据 包括文件所有者、访问权限、最后修改时间等
    public abstract void force(boolean meataData); 
    
    // 锁相关
    public final FileLock lock();
    public abstract FileLock lock(long position, long size, boolean shared); // shared:共享锁还是独占
    public final FileLocl tryLock();
    public abstract FileLock tryLock(long position, long size, boolean shared);
}
```

> 锁与文件关联，而不是与通道关联。使用锁来判优外部进程，而不是判优同一个java虚拟机上的线程。
>
> 文件锁用于进程 级别判优文件访问，比如程序组件之间或者集成其他供应商的组件时，而非用于多个java线程的并发访问。 

##### 使用

与RandomAccessFile：

```java
RandomAccessFile file = new RandomAccessFile("fileName", "r");
file.seek(1000); // set the file position
FileChannel channel = file.getChannel(); 
// channel.position() == 1000 -> RandomAccssFile 的 seek 会影响 FileChannel 的 position

channel.position(500);
// file.getFilePointer() == 500 -> channel 的 position 也会影响 RandomAccessFile 的 filePointer
```





#### SocketChannel

每个SocketChannel对象创建时都是同一对等的Socket对象串联，可以通过SocketChannel中的socket() 方法返回对应的Socket对象；同时在该Socket上调用getChannel() 能返回最初的SocketChannel对象；

反过来却不成立：直接创建的Socket对象不会关联SocketChannel对象，调用getChannel() 只会返回 null；

#####  使用例子

```java
InetSocketAddress addr = new InetSocketAddress("localhost", 8080);
SocketChannel sc = SocketChannel.oepn();
sc.configureBlocking(false);
sc.connect(addr);
while (!sc.finishConnect()) {
	// main work
}
sc.close();
```





#### 内存映射文件

##### 机制

- FileChannel 上调用 *map( )*方法会创建一个由磁盘文件支持的虚拟内存映射（virtual memory mapping），并在那块虚拟内存空间外部封装一个 *MappedByteBuffer* 对象。

- *map( )*方法返回的 *MappedByteBuffer* 对象的行为在多数方面类似一个基于内存的缓冲区，只不过该对象的数据元素存储在磁盘上的一个文件中。

- 调用 *get( )*方法会从磁盘文件中获取数据，此数据反映该文件的当前内容，即使在映射建立之后文件已经被一个外部进程做了修改。通过文件映射看到的数据同您用常规方法读取文件看到的内容是完全一样的。

##### 好处：比使用常规方法读写高效得多，甚至比使用通道的效率都高。

- 不需要做明确的系统调用；
- 操作系统的虚拟内存可以自动缓存内存页（memory page），所以不会消耗 Java 虚拟机内存堆（memory heap）；
- 一旦一个内存页已经生效（从磁盘上缓存进来），它就能以完全的硬件速度再次被访问而不需要再次调用系统命令来获取数据；

##### API

```java
public abstract class FileChannel extends AbstractChannel 
	   implements ByteChannel, GatheringByteChannel, ScatteringByteChannel {
    
	public abstract MappedByteBuffer map(MapMode mode, long position, long size);
    public static class MapMode {
    	public static final MapMode READ_ONLY;
    	public static final MapMode READ_WRITE;
    	public static final MapMode PRIVATE; // 写时拷贝
    }
}
// 映射整个文件
buffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
```

##### 注意

- 映射缓冲区没有绑定到创建它们的通道上。关闭相关联的 *FileChannel* 不会破坏映射，只有丢弃缓冲区对象本身才会破坏该映射；

- 所有的 *MappedByteBuffer* 对象都是直接的，这意味着它们占用的内存空间位于 Java 虚拟机内存堆之外；



##### MappedByteBuffer

```java
public abstract class MappedByteBuffer extends ByteBuffer {
	public final MappedByteByteBuffer load(); // 加载整个文件以使它常驻内存
	public boolean isLoaded();
	public final MappedByteBuffer force(); // 强制将映射缓冲区上的更改应用到永久磁盘存储器上
}
```

为文件创建虚拟内存映射之后，文件数据通常不会因此被从磁盘读取到内存（这取决于操作系统）。文件先被定位，然后一个文件句柄会被创建，当准备好之后就可以通过这个句柄来访问文件数据。



##### MappedByteBuffer & gather

```java
public class MappedHttp {
    private static final String INPUT_FILE = "MappedHttp.html";
    private static final String OUTPUT_FILE = "MappedHttp.out";

    private static final String LINE_SEP = "\r\n";
    private static final String SERVER_ID = "Server: Linux Server";
    private static final String HTTP_200_HDR = "HTTP/1.0 200 OK" + LINE_SEP + SERVER_ID + LINE_SEP;
    private static final String HTTP_404_HDR = "HTTP/1.0 404 Not Found" + LINE_SEP + SERVER_ID + LINE_SEP;
    private static final String MSG_404 = "Could not open file: ";

    // 读取文件 以HTTP形式写入另一文件
    public static void main(String[] argv) throws Exception {
        ByteBuffer header = ByteBuffer.wrap(bytes(HTTP_200_HDR));
        ByteBuffer body = ByteBuffer.allocate(128);
        ByteBuffer[] gather = {header, body, null};

        String contentType = "unknown/unknown";
        long contentLength = -1;

        try {
            FileInputStream fis = new FileInputStream(INPUT_FILE);
            FileChannel fc = fis.getChannel();
            MappedByteBuffer fileData = fc.map(FileChannel.MapMode.READ_ONLY, 0, fc.size());
            gather[2] = fileData;
            contentLength = fc.size();
            contentType = URLConnection.guessContentTypeFromName(INPUT_FILE);
        } catch (IOException e) {
            // report problem when file could not be open
            ByteBuffer buf = ByteBuffer.allocate(128);
            String msg = MSG_404 + e + LINE_SEP;
            buf.put(bytes(msg));
            buf.flip();
            // use http error response
            gather[0] = ByteBuffer.wrap(bytes(HTTP_404_HDR));
            gather[2] = buf;
            contentLength = msg.length();
            contentType = "text/plain";
        }
        String sb = "Content-Length: " + contentLength +
                LINE_SEP +
                "Content-Type: " + contentType +
                LINE_SEP + LINE_SEP;
        body.put(bytes(sb));
        body.flip();

        FileOutputStream fos = new FileOutputStream(OUTPUT_FILE);
        FileChannel out = fos.getChannel();
        while (out.write(gather) > 0) {
            // empty
        }
        out.close();
        System.out.println("output to: " + OUTPUT_FILE);
    }

    private static byte[] bytes(String string) throws Exception{
        return (string.getBytes (StandardCharsets.US_ASCII));
    }
}
```

