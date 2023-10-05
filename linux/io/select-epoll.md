### select

#### 函数原型

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

1. 为fd是⼀个int值，所以fd_set其实是⼀个bit数组，每1位表⽰⼀个fd是否有读事件或者写事件发⽣；
2. 第⼀个参数是readfds或者writefds的下标的最⼤值+1。因为fd从0开始，+1才表⽰个数；
3. 返回结果还在readfds和writefds⾥⾯，操作系统会重置所有的bit位，告知应⽤程序到底哪个fd上⾯有事件，应⽤程序需要⾃⼰从0到maxfds-1遍历所有的fd，然后执⾏相应的read/write操作；
4. 每次当select调⽤返回后，在下⼀次调⽤之前，要重新维护readfds和writefds；





#### 使用

##### 添加被检测的文件描述符集合

```cpp
#include <sys/select.h>

fd_set read_fds;   // 读文件描述符集合
fd_set write_fds;  // 写文件描述符集合
fd_set except_fds; // 异常文件描述符集合

FD_ZERO(&read_fds);
FD_ZERO(&write_fds);
FD_ZERO(&except_fds);

// 将需要监听的文件描述符添加到对应的集合中
FD_SET(your_file_descriptor, &read_fds);   // 添加读文件描述符
FD_SET(another_file_descriptor, &write_fds);  // 添加写文件描述符
```

##### 调用 select 

```cpp
// 添加超时时间 5s
struct timeval timeout;
timeout.tv_sec = 5;       
timeout.tv_usec = 0;

int max_fd = your_max_file_descriptor + 1;  // 最大文件描述符加一

int num_ready = select(max_fd, &read_fds, &write_fds, &except_fds, &timeout);
if (num_ready == -1) {
    // select 出错，处理错误
} else if (num_ready == 0) {
    // 超时，没有事件发生
} else {
    // 处理发生的事件
    if (FD_ISSET(your_file_descriptor, &read_fds)) {
        // 可读事件发生，读取数据
        // 处理读事件
    }
    if (FD_ISSET(another_file_descriptor, &write_fds)) {
        // 可写事件发生，写入数据
        // 处理写事件
    }
}
```

注意事项：

- 在使用 `select` 之前，需要确保文件描述符处于非阻塞模式，以免阻塞 `select` 调用。
- `select` 函数是阻塞的，直到超时或有事件发生才返回，可以通过设置超时时间为 0 实现非阻塞调用。
- `select` 函数返回后，可以使用 `FD_ISSET` 宏来检查具体的文件描述符是否就绪。
- `select` 函数有文件描述符数量的限制，通常在较小规模的应用中使用较为合适

#### 工作原理

1. 将文件描述符集合拷贝到内核：
   在调用 `select` 之前，需要将关注的文件描述符集合通过参数传递给 `select` 函数。这样，`select` 函数可以将这些文件描述符集合的信息拷贝到内核中，以便内核在后续进行事件检测时使用。
2. 内核进行事件检测：
   当调用 `select` 函数后，它会阻塞等待，直到超时时间到达或有文件描述符就绪。此时，内核会检查所关注的文件描述符集合，判断其中的文件描述符是否有可读、可写或异常等事件发生。
3. 返回就绪的文件描述符：
   当有文件描述符就绪时，`select` 函数会返回，通知调用者哪些文件描述符可以进行读取、写入或其他处理。可以使用 `FD_ISSET` 宏来检查哪些文件描述符处于就绪状态。

##### 内核如何得知文件描述符就绪

1. 轮询（Polling）：
   在早期的实现中，内核使用轮询机制来监视文件描述符的就绪事件。它会遍历所有关注的文件描述符，并检查每个文件描述符的状态。这种方式效率较低，因为需要不停地遍历文件描述符的集合，但在某些情况下仍然被使用。
2. 中断（Interrupt）：
   在某些操作系统中，内核使用中断机制来监视文件描述符的状态变化。当文件描述符就绪时，内核会触发一个中断信号，通知进程有事件发生。进程可以通过捕获中断信号来得知文件描述符的就绪状态。
3. 事件驱动（Event-driven）：
   现代操作系统通常采用事件驱动的方式来监视文件描述符的就绪事件。这种方式使用了更高效的机制，如 epoll（Linux）、kqueue（FreeBSD、Mac OS X）或 IOCP（Windows）等。这些机制允许内核维护一个事件队列，只有在文件描述符就绪时才会向进程报告。进程可以通过等待事件通知或异步回调等方式处理就绪的文件描述符。



### poll

#### 函数原型

```c
#include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);

struct pollfd {
    int fd;       // 文件描述符
    short events; // 要监视的事件
    short revents; // 就绪的事件
};
// fd：要监视的文件描述符
// events：要监视的事件，可以是以下值的按位或组合：
//    POLLIN：可读事件。
//    POLLOUT：可写事件。
//    POLLERR：错误事件。
//    POLLHUP：挂起事件。
//    POLLNVAL：无效事件。
// revents：就绪的事件，由 poll 函数填充，表示实际发生的事件。
```

- `fds`：指向 `pollfd` 结构体数组的指针，每个结构体表示一个要监视的文件描述符和所需的事件。
- `nfds`：`fds` 数组中元素的数量。
- `timeout`：超时时间，以毫秒为单位，设置为 `-1` 表示无限等待，设置为 `0` 表示立即返回，设置为正数表示等待指定的毫秒数。

#### 使用

```c
#include <stdio.h>
#include <poll.h>
#include <unistd.h>

int main() {
    struct pollfd fds[2];
    fds[0].fd = STDIN_FILENO;   // 监视标准输入
    fds[0].events = POLLIN;
    fds[1].fd = STDOUT_FILENO;  // 监视标准输出
    fds[1].events = POLLOUT;

    while (1) {
        int ready = poll(fds, 2, -1);
        if (ready == -1) {
            perror("poll");
            return 1;
        }

        for (int i = 0; i < 2; i++) {
            if (fds[i].revents & POLLIN) {
                if (fds[i].fd == STDIN_FILENO) {
                    // 标准输入就绪，可以进行读取操作
                    char buffer[256];
                    fgets(buffer, sizeof(buffer), stdin);
                    printf("输入：%s", buffer);
                }
            }

            if (fds[i].revents & POLLOUT) {
                if (fds[i].fd == STDOUT_FILENO) {
                    // 标准输出就绪，可以进行写入操作
                    printf("输出：Hello, World!\n");
                }
            }
        }
    }

    return 0;
}
```



#### 相较于 select

`poll` 函数的优势之一是不需要在每次调用之前重新设置需要监视的文件描述符，而是通过 `pollfd` 结构体的 `revents` 字段来获取就绪的事件。





### epoll

#### 使用

##### 创建 epoll 实例

```cpp
#include <sys/epoll.h>
int epoll_fd = epoll_create1(0);
if (epoll_fd == -1) {
    // 创建 epoll 实例失败，处理错误
}
```

##### 添加事件到 epoll实例

```cpp
struct epoll_event event;
event.events = EPOLLIN;  // 设置事件类型，例如读事件
event.data.fd = your_file_descriptor;  // 设置关联的文件描述符

if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, your_file_descriptor, &event) == -1) {
    // 添加事件失败，处理错误
}
```

##### 等待事件发生

```cpp
const int max_events = 10;
struct epoll_event events[max_events];

int num_events = epoll_wait(epoll_fd, events, max_events, timeout);
if (num_events == -1) {
    // epoll_wait 发生错误，处理错误
} else if (num_events == 0) {
    // 超时，没有事件发生
} else {
    // 处理发生的事件
    for (int i = 0; i < num_events; ++i) {
        if (events[i].events & EPOLLIN) {
            // 可读事件发生，读取数据
            int fd = events[i].data.fd;
            // 处理读事件
        }
        // 其他事件类型的处理，如 EPOLLOUT、EPOLLERR 等
    }
}
```

##### 修改数据类型

```cpp
// 修改事件
struct epoll_event event;
event.events = EPOLLIN | EPOLLOUT;  // 修改事件类型
event.data.fd = your_file_descriptor;

if (epoll_ctl(epoll_fd, EPOLL_CTL_MOD, your_file_descriptor, &event) == -1) {
    // 修改事件失败，处理错误
}

// 删除事件
if (epoll_ctl(epoll_fd, EPOLL_CTL_DEL, your_file_descriptor, nullptr) == -1) {
    // 删除事件失败，处理错误
}
```

##### 注意事项：

- 使用 epoll 前需要确保文件描述符处于非阻塞模式，可以使用 `fcntl` 函数将其设置为非阻塞模式。
- epoll 实例的创建和事件的添加、修改、删除都可以在任意线程中进行，但等待事件发生的 epoll_wait 函数通常在主线程中调用。



> event.data.fd = your_file_descriptor; // 设置关联的文件描述符
>
> if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, your_file_descriptor, &event) == -1) {
> // 添加事件失败，处理错误
> }
> 为什么已经在 event中设置了文件描述符，但是还需要在 epoll_ctl 中既传入 event 还要传入文件描述符，而不是从event中直接获取文件描述符

回答：

原因是为了保持与之前的版本兼容性，以及提供更多的灵活性。在早期的实现中，`epoll_event` 结构可能具有其他字段，而不仅仅是 `events` 和 `data` 字段。通过传递文件描述符作为参数，可以确保在不同的内核版本和实现中具有一致的行为。

此外，将文件描述符作为参数传递给 `epoll_ctl` 函数还可以提高效率。内核可以更快速地定位要操作的 epoll 实例，并进行相应的操作，而不需要在每次调用 `epoll_ctl` 时都从 `event` 结构中提取文件描述符。



#### 触发

##### 水平触发（LT）：条件触发

读缓冲区只要不为空，就会⼀直触发读事件；写缓冲区只要不满，就会⼀直触发写事件；

**注意**：要避免“写的死循环”问题：写缓冲区为满的概率很⼩，即“写的条件”会⼀直满⾜，所以当⽤户注册了写事件却没有数据要写时，它会⼀直触发，因此在 LT 模式下写完数据⼀定要取消写事件。



##### 边缘触发（ET）：状态触发

读缓冲区的状态，从空转为⾮空的时候触发⼀次；写缓冲区的状态，从满转为⾮满的时候触发⼀次；

**注意**：要避免“short read”问题：例如⽤户收到100个字节，它触发1次，但⽤户只读到了50个字节，剩下的50个字节不读，它也不会再次触发。因此在ET模式下，⼀定要把“读缓冲区”的数据⼀次性读完。



#### 总结

##### 过程

 整个过程大致分为三个部分：

1. 事件注册。通过函数epoll_ctl实现。对于服务器⽽⾔，是accept、read、write 三种事件；对于客户端⽽⾔，是connect、read、write三种事件；
2. 轮询这三个事件是否就绪。通过函数epoll_wait实现。有事件发⽣，该函数返回；
3. 事件就绪，执⾏实际的I/O操作。通过函数accept/read/write实现；
   - read事件就绪：这个很好理解，是远程有新数据来了，socket读取缓存区⾥有数据，需要调⽤read函数处理；
   - write事件就绪：是指本地的socket写缓冲区是否可写。如果写缓冲区没有满，则⼀直是可写的，write事件⼀直是就绪的，可以调⽤write函数；
   - accept事件就绪：有新的连接进⼊，需要调⽤accept函数处理；



##### 触发

在实际开发中，⼤家⼀般都倾向于⽤LT，这也是默认的模式，JavaNIO⽤的也是epoll的LT模式。

因为ET容易漏事件，⼀次触发如果没有处理好，就没有第⼆次机会了。虽然LT重复触发可能有少许的性能损耗，但代码写起来更安全。



##### 应用：1 + N + M 模式

- 1: 一个监听线程；负责accept事件的注册和处理；
  - 每⼀个新进来的客户端建⽴socket连接；
  - 然后把socket连接移交给I/O线程，完成任务，继续监听新的客户端；
- N：n个io线程；（通常等于CPU核数）
  - 负责每个socket连接上⾯read/write事件的注册和实际的socket的读写；
  - 把读到的Reqeust放⼊Request队列，交由Worker线程处理；
- M：m个worker 线程；业务线程，没有socket读写；
  1. 对Request队列进⾏处理，⽣成Response队列；
  2. 然后将处理结果写⼊Response队列；
  3. 由I/O线程再回复给客户端；



总之：

I/O线程监听到⼀个sokcet连接上有读事件，于是把socket移交给Worker线程，Woker线程读出数据，处理完业务逻辑，直接返回给客户端；

之所以可以这么做，是因为I/O线程已经检测到读事件就绪，所以当Worker线程在读的时候不会等待；









