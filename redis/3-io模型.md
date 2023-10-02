#### 阻塞模型

##### 工作过程

1. 监听客户端请求（bind/listen）
2. 客户端建立连接（accept）
3.  socket 中读取请求（recv）
4. 解析客户端发送请求（parse）
5. 根据请求类型读取键值数据（get）
6. 最后给客户端返回结果，即向 socket 中写回数据（send）

其中：

- accept：监听到一个客户端有连接请求，但一直未能成功建立起连接时，会阻塞在 accept() 函数，导致其他客户端无法和 Redis 建立连接；
- recv：当 Redis 通过 recv() 从一个客户端读取数据时，如果数据一直没有到达，Redis 也会一直阻塞在 recv()；



##### accept 示例

```cpp
#include <iostream>
#include <sys/socket.h>
#include <arpa/inet.h>

int main() {
    int serverSocket;
    int clientSocket;
    struct sockaddr_in serverAddress, clientAddress;
    
    // 创建Socket
    serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    
    // 设置服务器地址
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_port = htons(8080);  // 设置监听的端口号
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    
    // 绑定Socket到服务器地址
    bind(serverSocket, (struct sockaddr*)&serverAddress, sizeof(serverAddress));
    
    // 监听连接请求
    listen(serverSocket, 5);  // 设置允许同时连接的最大客户端数量
    
    std::cout << "Server listening on port 8080..." << std::endl;
    
    // 循环接受客户端连接请求
    while (true) {
        socklen_t clientAddressSize = sizeof(clientAddress);
        
        // 接受客户端连接请求
        clientSocket = accept(serverSocket, (struct sockaddr*)&clientAddress, &clientAddressSize);
        
        // 处理客户端连接...
        
        // 关闭客户端Socket
        close(clientSocket);
    }
    
    // 关闭服务器Socket
    close(serverSocket);
    
    return 0;
}
```

注：

> `accept()`函数：
>
> - `accept()`函数用于在服务器端接受客户端的连接请求。
> - 当服务器处于监听状态时，通过调用`accept()`函数可以接受客户端的连接，并创建一个新的Socket描述符来与该客户端进行通信。
> - `accept()`函数会阻塞当前线程，直到有新的连接请求到达并被接受;
>
> 
>
> `select()`函数：
>
> - `select()`函数用于在多个Socket描述符上进行I/O多路复用。
> - 它允许程序同时监视多个Socket描述符上的读写事件，并在有事件发生时进行处理。
> - `select()`函数接受三个用于描述Socket集合的参数，分别是可读Socket集合、可写Socket集合和异常Socket集合。
> - 当其中任何一个Socket描述符有可读、可写或异常事件发生时，`select()`函数会返回，并告知哪些Socket描述符发生了事件。
> - `select()`函数可以设置超时时间，若在指定时间内没有任何事件发生，则返回。





#### 非阻塞模式

##### accept

当 Redis 调用 accept() 但一直未有连接请求到达时，Redis 线程可以返回处理其他操作，而不用一直等待

```cpp
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>

int main() {
    int serverSocket, clientSocket;
    struct sockaddr_in serverAddress, clientAddress;
    
    // 创建 Socket 并设置服务器地址...
    
    // 将 Socket 设置为非阻塞模式
    int flags = fcntl(serverSocket, F_GETFL, 0);
    fcntl(serverSocket, F_SETFL, flags | O_NONBLOCK);
    
    // 监听连接请求...
    
    // 在循环中接受客户端连接请求
    while (true) {
        socklen_t clientAddressSize = sizeof(clientAddress);
        
        // 非阻塞地接受客户端连接请求
        clientSocket = accept(serverSocket, (struct sockaddr*)&clientAddress, &clientAddressSize);
        
        if (clientSocket == -1) {
            // 检查是否是非阻塞模式下没有新的连接请求
            if (errno == EWOULDBLOCK || errno == EAGAIN) {
                // 没有新的连接请求，继续循环或进行其他操作
                continue;
            } else {
                // 其他错误处理
                std::cerr << "Error accepting client connection" << std::endl;
                break;
            }
        }
        
        // 处理客户端连接...
        
        // 关闭客户端 Socket
        close(clientSocket);
    }
    
    // 关闭服务器 Socket..
    return 0;
}
```



##### recv

配置成非阻塞

```cpp
// 将 Socket 设置为非阻塞模式
int flags = fcntl(clientSocket, F_GETFL, 0);
fcntl(clientSocket, F_SETFL, flags | O_NONBLOCK);

char buffer[1024];
// 非阻塞地接收客户端数据
ssize_t bytesRead = recv(clientSocket, buffer, sizeof(buffer), 0);
if (bytesRead == -1) {
    // 检查是否是非阻塞模式下没有可读数据
    if (errno == EWOULDBLOCK || errno == EAGAIN) {
        // 没有可读数据，继续循环或进行其他操作
        continue;
    } else {
        // 其他错误处理
        std::cerr << "Error receiving data from client" << std::endl;
        break;
    }
} else if (bytesRead == 0) {
    // 客户端关闭连接
    std::cout << "Client disconnected" << std::endl;
    break;
}
```



#### IO 多路复用

指一个线程处理多个 IO 流，即select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，**该机制允许内核中，同时存在多个监听套接字和已连接套接字**。

内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

![img](https://static001.geekbang.org/resource/image/00/ea/00ff790d4f6225aaeeebba34a71d8bea.jpg)

