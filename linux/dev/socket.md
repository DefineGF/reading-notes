网络数据流采用大端字节序列；很多计算机采用小端存储。

```cpp
#include <arpa/inet.h>

// 主机字节序列 -> 网络字节序列
uint16_t htons(uint16_t hostshort);
uint32_t htonl(uint32_t hostlong);

// 网络序列 -> 主机序列
uint16_t ntohs(uint16_t netshort);
uint32_t ntohl(uint32_t netlong);

// ip字符串形式（127.0.0.1） -> int (大端)
int inet_pton(int af, const char *src, void *dst);

// 网络字节序int  -> 字符串
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```

##### int socket(int domain, int type, int protocol);

*   domin：AF\_INET - ipv4；AF\_INET6 - ipv6;
*   type：tcp - 流式协议  udp - 报式协议;
*   protocol: 协议类型；
*   返回值：文件描述符(套接字)

##### int bind(int sockfd, const struct sockaddr \*addr, socklen\_t addrlen);

*   sockfd  - 创建出的文件描述符
*   addr    - 绑定协议、端口、IP地址
*   addrlen - addr结构体的长度

sockaddr 结构体：

```cpp
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};

struct in_addr {
    uint32_t       s_addr;     /* address in network byte order */
};
```

##### int listen(int sockfd, int backlog);

*   socket 函数创建出来的文件描述符
*   backlog - 最大值 128

##### select

```c
int select(int nfds, 
            fd_set *readfds, 
            fd_set *writefds, 
            fd_set *exceptfds, 
            struct timeval *timeout);
            
// 将一个给定的文件描述符从集合中删除
void FD_CLR(int fd, fd_set *set);	

// 检查集合中指定的文件描述符是否可以读、写、有发生异常
int  FD_ISSET(int fd, fd_set *set);		

// 将一个给定的文件描述符加入集合之中
void FD_SET(int fd, fd_set *set);	

// 清空集合
void FD_ZERO(fd_set *set);				
```

*   nfds：一个整数值，是指集合中所有文件描述符的范围，即所有文件描述符的最大值加1;
*   readfds：指向fd\_set结构的指针，监视这些文件描述符的读变化，如果这个集合中有一个文件可读，select就会返回一个大于0的值，表示有文件可读；如果没有可读的文件，则根据timeout参数再判断是否超时，若超出timeout的时间，select返回0，若发生错误返回负值。可以传入NULL值，表示不关心任何文件的读变化;
*   writefds：指向fd\_set结构的指针，监视这些文件描述符的写变化的，如果这个集合中有一个文件可写，select就会返回一个大于0的值，表示有文件可写，如果没有可写的文件，则根据timeout参数再判断是否超时，若超出timeout的时间，select返回0，若发生错误返回负值。可以传入NULL值，表示不关心任何文件的写变化;
*   exceptfds：上面两个参数的意图，用来监视文件错误异常文件;
*   timeout：是select的超时时间，这个参数至关重要，它可以使select处于三种状态:
    *   NULL: 将select置于阻塞状态，一定等到监视文件描述符集合中某个文件描述符发生变化为止；
    *   0: 变成一个纯粹的非阻塞函数，不管文件描述符是否有变化，都立刻返回继续执行，文件无变化返回0，有变化返回一个正值；
    *   大于0，即select在timeout时间内阻塞，超时时间之内有事件到来就返回了，否则在超时后不管怎样一定返回，返回值同上述。
*   函数返回：
    *   当监视的相应的文件描述符集中满足条件时，比如说读文件描述符集中有数据到来时，内核(I/O)根据状态修改文件描述符集，并返回一个大于0的数。
    *   当没有满足条件的文件描述符，且设置的timeval监控时间超时时，select函数会返回一个为0的值。
    *   发生错误, 返回负值

##### int accept(int sockfd, struct sockaddr \*addr, socklen\_t \*addrlen);

*   sockfd:文件描述符, 使用socket创建出的文件描述符
*   addr: 存储客户端的端口和IP, 传出参数
*   addrlen: 传入传出参数
*   返回值: 返回的是一个套接字,

对应客户端: 服务器端与客户端进程通信使用accept的返回值对应的套接字

##### int connect(int sockfd, const struct sockaddr \*addr, socklen\_t addrlen);

*   sockfd: 套接字
*   addr: 服务器端的IP和端口
*   addrlen: 第二个参数的长度

##### 示例1

server：

```cpp
#include <iostream>
#include <stdio.h>
#include <cstring>       // void *memset(void *s, int ch, size_t n);
#include <sys/types.h>   // 数据类型定义
#include <sys/socket.h>  // 提供socket函数及数据结构sockaddr
#include <arpa/inet.h>   // 提供IP地址转换函数，htonl()、htons()...
#include <netinet/in.h>  // 定义数据结构sockaddr_in
#include <ctype.h>       // 小写转大写
#include <unistd.h>      // close()、read()、write()、recv()、send()...
using namespace std;

const int flag = 0; // 0表示读写处于阻塞模式
const int port = 8080;
const int buffer_size = 1<<20;


int main(int argc, const char* argv[]){
    // 创建服务器监听的套接字。Linux下socket被处理为一种特殊的文件，返回一个文件描述符。
    // int socket(int domain, int type, int protocol);
    // domain设置为AF_INET/PF_INET，即表示使用ipv4地址(32位)和端口号（16位）的组合。
    int server_sockfd = socket(PF_INET, SOCK_STREAM, 0);  
    if(server_sockfd == -1){
        close(server_sockfd);
        perror("socket error!");
    }
    // /* Enable address reuse */
    // int on = 1;
    // int ret = setsockopt( server_sockfd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on) );

    // 此数据结构用做bind、connect、recvfrom、sendto等函数的参数，指明地址信息。
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr)); // 结构体清零
    server_addr.sin_family = AF_INET;    // 协议
    server_addr.sin_port = htons(port);  // 端口16位, 此处不用htons()或者错用成htonl()会连接拒绝!!
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY); // 本地所有IP
    // 另一种写法, 假如是127.0.0.1
    // inet_pton(AF_INET, "127.0.0.1", &server_addr.sin_addr.s_addr);


    // int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen); 
    // bind()函数的主要作用是把ip地址和端口绑定到套接字(描述符)里面
    // struct sockaddr是通用的套接字地址，而struct sockaddr_in则是internet环境下套接字的地址形式，二者长度一样，都是16个字节。二者是并列结构，指向sockaddr_in结构的指针也可以指向sockaddr。
    // 一般情况下，需要把sockaddr_in结构强制转换成sockaddr结构再传入系统调用函数中。
    if(bind(server_sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1){
        close(server_sockfd);
        perror("bind error");
    }
    // 第二个参数为相应socket可以排队的准备道来的最大连接个数
    if(listen(server_sockfd, 5) == -1){
        close(server_sockfd);
        perror("listen error");
    }

    printf("Listen on port %d\n", port);
    while(1){
        struct sockaddr_in client_addr;
        socklen_t client_len = sizeof(client_addr);
        
        // accept()函数从处于established状态的连接队列头部取出一个已经完成的连接，
        // 如果这个队列没有已经完成的连接，accept()函数就会阻塞当前线程，直到取出队列中已完成的客户端连接为止。
        int client_sockfd = accept(server_sockfd, (struct sockaddr*)&client_addr, &client_len);

        char ipbuf[128];
        printf("client iP: %s, port: %d\n", inet_ntop(AF_INET, &client_addr.sin_addr.s_addr, ipbuf, sizeof(ipbuf)),
            ntohs(client_addr.sin_port));

        // 实现客户端发送小写字符串给服务端，服务端将小写字符串转为大写返回给客户端
        char buf[buffer_size];
        while(1) {
            // read data, 阻塞读取
            int len = recv(client_sockfd, buf, sizeof(buf),flag);
            if (len == -1) {
                close(client_sockfd);
                close(server_sockfd);
                perror("read error");
            }else if(len == 0){  // 这里以len为0表示当前处理请求的客户端断开连接
                break;
            }
            printf("read buf = %s", buf);
            // 小写转大写
            for(int i=0; i<len; ++i) {
                buf[i] = toupper(buf[i]);
            }
            printf("after buf = %s", buf);

            // 大写串发给客户端
            if(send(client_sockfd, buf, strlen(buf),flag) == -1){
                close(client_sockfd);
                close(server_sockfd);
                perror("write error");
            }
            memset(buf,'\0',len); // 清空buf
        }
        close(client_sockfd);
    }
    close(server_sockfd);

    return 0;
}
```

client

```cpp
// client 端相对简单, 另外可以使用nc命令连接->nc ip prot
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <string.h>


const int port = 8080;
const int buffer_size = 1<<20;
int main(int argc, const char *argv[]) {
    

    int client_sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (client_sockfd == -1) {
        perror("socket error");
        exit(-1);
    }
    

    struct sockaddr_in client_addr;
    bzero(&client_addr, sizeof(client_addr));
    client_addr.sin_family = AF_INET;
    client_addr.sin_port = htons(port);
    inet_pton(AF_INET, "127.0.0.1", &client_addr.sin_addr.s_addr);
    
    int ret = connect(client_sockfd, (struct sockaddr*)&client_addr, sizeof(client_addr));
    if (ret == -1) {
        perror("connect error");
        exit(-1);
    }
    
    while(1) {
        char buf[buffer_size] = {0};
        fgets(buf, sizeof(buf), stdin); // 从终端读取字符串
        write(client_sockfd, buf, strlen(buf));
        //接收, 阻塞等待
        int len = read(client_sockfd, buf, sizeof(buf));
        if (len == -1) {
             perror("read error");
             exit(-1);
        }
        printf("client recv %s\n", buf);
        
    }
    
    close(client_sockfd);
    return 0;
}
```

##### 示例2

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <ctype.h>

#define SERV_PORT 6666

int main(int argc, char *argv[])
{
    int listenfd, connfd;
    char buf[BUFSIZ], str[INET_ADDRSTRLEN];

    struct sockaddr_in clie_addr, serv_addr;
    socklen_t clie_addr_len;

    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    int opt = 1;
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt)); //SO_REUSEADDR允许重用本地地址端口

    bzero(&serv_addr, sizeof(serv_addr));

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY); // 绑定本地IP地址
    serv_addr.sin_port = htons(SERV_PORT);		   // 设置端口

    bind(listenfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    listen(listenfd, 128);

    fd_set rset, allset;
    int ret, maxfd = 0;
    maxfd = listenfd;
    FD_ZERO(&allset);           // 集合全部设为0
    FD_SET(listenfd, &allset);  // 将listenfd添加到文件描述符集合

    while (1)
    {
        rset = allset;
        ret = select(maxfd + 1, &rset, NULL, NULL, NULL); // 监听rset 读事件，返回有事件发生的个数
        if (FD_ISSET(listenfd, &rset))
        {
            clie_addr_len = sizeof(clie_addr);
            connfd = accept(listenfd, (struct sockaddr *)&clie_addr, &clie_addr_len);
            printf("received from %s at PORT %d\n", inet_ntop(AF_INET, &clie_addr.sin_addr, str, sizeof(str)), ntohs(clie_addr.sin_port));

            FD_SET(connfd, &allset); //将一个集合中的事件“加入”
            if (maxfd < connfd)
                maxfd = connfd;
            if (ret == 1)
                continue; //说明select只返回一个，并且只是listenfd，无需后续执行
        }
        int n = 0; // read读到的字节数
        for (int i = listenfd + 1; i < maxfd + 1; ++i)
        {
            if (FD_ISSET(i, &rset)) //判断文件描述符是否在集合中
            {
                n = read(i, buf, sizeof(buf));
                if (n == 0)
                {
                    close(i);
                    FD_CLR(i, &allset); //将一个集合中的事件“拿走”
                }
                else if (n == -1)
                {
                }
                else
                    for (int j = 0; j < n; ++j)
                        buf[j] = toupper(buf[j]);
                write(i, buf, n);
                write(STDOUT_FILENO, buf, n);
            }
        }
    }

    close(listenfd);
    return 0;
}
```

