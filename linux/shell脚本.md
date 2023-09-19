#### 启动 | 关闭 jar 

```shell
#!/bin/bash

# 定义JAR包的路径和名称
JAR_FILE="/path/to/your/jarfile.jar"

# 定义启动命令
start() {
    echo "Starting the application..."
    java -jar "$JAR_FILE" &
}

# 定义停止命令
stop() {
    echo "Stopping the application..."
    # 查找并杀死与JAR包相关的进程
    PID=$(ps aux | grep "$JAR_FILE" | grep -v grep | awk '{print $2}')
    if [ -n "$PID" ]; then
        kill "$PID"
        echo "Application stopped."
    else
        echo "Application is not running."
    fi
}

# 根据传入的参数执行对应的命令
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
        ;;
esac
```

运行：

> chmod +x shell_name.sh
>
> ./shell_name.sh start

##### stop

ps aux:

- `ps` 是 "process status" 的缩写，用于获取进程状态信息;
- `a` 选项表示显示所有用户的进程，而不仅仅是当前用户的进程;
- `u` 选项表示以详细格式显示进程信息，包括进程的用户、CPU占用、内存占用等

示例：

```shell
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.4 168452  8556 ?        Ss   Sep16   0:07 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Sep16   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   Sep16   0:00 [rcu_gp]
```

- `USER`：进程所属的用户。
- `PID`：进程ID。
- `%CPU`：该进程使用的CPU百分比。
- `%MEM`：该进程使用的内存百分比。
- `VSZ`：进程的虚拟内存大小（以KB为单位）。
- `RSS`：进程的实际内存大小（以KB为单位）。
- `TTY`：进程关联的终端设备。
- `STAT`：进程的状态。
- `START`：进程的启动时间。
- `TIME`：进程的累计CPU使用时间。
- `COMMAND`：启动该进程的命令。



grep:

- -v 反向过滤

awk 'print {'$2}'

- awk：按行处理；
- 