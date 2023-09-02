函数声明：

##### void accept_request(int)



#### 流程

##### main

1. 首先通过 startup 函数绑定相关端口进行监听；
2. 然后通过 accept 接收客户端的连接；
3. 有客户端连接时，创建子进程，执行 accept_request 逻辑；
4. 最后关闭监听端口

```c
int client_sock = -1;
struct sockaddr_in client_name;
int client_name_len = sizeof(client_name);
pthread_t newthread;

u_short port = 0;       // 端口号
int server_sock = -1;   // 
server_sock = startup(&port); // 在相应端口创建 服务 
printf("httpd running on port %d\n", port);

while (1)
{
    client_sock = accept(server_sock, (struct sockaddr *)&client_name, &client_name_len);
    if (client_sock == -1)
        error_die("accept");
    if (pthread_create(&newthread , NULL, accept_request, client_sock) != 0)
        perror("pthread_create");
}
close(server_sock);
```



##### 1. int startup(u_short *port)

1. 创建监听 socket，并初始化：设置协议、端口、ip地址等；（如果初始设置端口为0，则随机生成端口号）
2. 通过 bind 函数完成 监听socket 和 sockaddr_in 的绑定，并开启监听；
3. 返回监听socket的文件定义符；（int）

```c
int httpd = 0;
struct sockaddr_in name;

/*建立 socket */
httpd = socket(PF_INET, SOCK_STREAM, 0);
if (httpd == -1)
    error_die("socket");
memset(&name, 0, sizeof(name));
name.sin_family = AF_INET;
name.sin_port = htons(*port);
name.sin_addr.s_addr = htonl(INADDR_ANY);

if (bind(httpd, (struct sockaddr *)&name, sizeof(name)) < 0)
    error_die("bind");
/*如果当前指定端口是 0，则动态随机分配一个端口*/
if (*port == 0)  /* if dynamically allocating a port */
{
    int namelen = sizeof(name);
    if (getsockname(httpd, (struct sockaddr *)&name, &namelen) == -1)
        error_die("getsockname");
    *port = ntohs(name.sin_port);
}
/*开始监听*/
if (listen(httpd, 5) < 0)
    error_die("listen");
/*返回 socket id */
return(httpd);
```



##### 2. void accept_request(int client)

> 注：http请求头格式如下
>
> GET /home.html HTTP/1.1
> Host: developer.mozilla.org
> User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
> Accept-Language: en-US,en;q=0.5
> Accept-Encoding: gzip, deflate, br
> Referer: https://developer.mozilla.org/testpage.html
> Connection: keep-alive
> Upgrade-Insecure-Requests: 1
> If-Modified-Since: Mon, 18 Jul 2016 02:36:04 GMT
> If-None-Match: "c561c68d0ba92bbeb8b0fff2a9199f722e3a621a"
> Cache-Control: max-age=0

本项目中：

>GET / HTTP/1.1
>
>Host: 127.0.0.1:4000
>
>User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/111.0
>
>Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
>
>Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
>
>Accept-Encoding: gzip, deflate, br
>
>Connection: keep-alive
>
>Upgrade-Insecure-Requests: 1
>
>Sec-Fetch-Dest: document
>
>Sec-Fetch-Mode: navigate
>
>Sec-Fetch-Site: none
>
>Sec-Fetch-User: ?1





1. 首先请求第一行，获取请求方法：POST | GET；





