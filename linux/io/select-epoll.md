### select

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