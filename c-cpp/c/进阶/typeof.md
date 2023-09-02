#### typeof 用法

##### 获取函数返回类型

例子一

```c
int *get_int_info()
{
    int *data = malloc(sizeof(int));
    *data = 1024;
    return data;
}
```

使用：

```c
typeof(get_int_info()) i1;  // 获取类型
i1 = get_int_info();        // 调用函数
```



例子二：复杂类型

```c
struct apple
{
    int weight;
    int color;
};

struct apple *get_apple_info()
{
    struct apple *a1;
    a1 = malloc(sizeof(struct apple));
    a1->weight = 12;
    a1->color = 1;
    return a1;
}
```

使用：

```c
typeof(get_apple_info()) r1;
r1 = get_apple_info();
printf("get weight = %d, color = %d\n", r1->weight, r1->color);
```



##### 获取参数类型

```c
#define max(x, y) (         \
    {                       \
        typeof(x) _x = (x); \
        typeof(y) _y = (y); \
        (void)(&_x == &_y); \
        _x > _y ? _x : _y;  \
    })
```

> 注意：第5行。如果两个参数不是同一种类型的话，编译器会报错。

正确使用：

```c
int a = 3, b = 4;
int _max = max(a, b);
```

编译和运行成功！但是以下编译错误：

```c
float c = 5.0f;
max(a, c);
```

