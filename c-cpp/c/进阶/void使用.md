

##### 类型转换

```c
typedef enum data_type_
{
    CHAR = 1,
    INT = 2,
    FLOAT = 3,
    DOUBLE = 4
} Data_Type;

void cast(Data_Type data_type, void *data)
{
    if (data_type == INT)
    {
        int *data_int = (int *)data;
        printf("get the value_int is: %d\n", *data_int);
    }
    else if (data_type == CHAR)
    {
        char *data_char = (char *)data;
        printf("get the value_char is: %c\n", *data_char);
    }
    else if (data_type == DOUBLE)
    {
        double *data_dobule = (double *)data;
        printf("get the value_double is: %lf\n", *data_dobule);
    }
}
```

需要指出的是，使用 void* 指向的数据回收时，需要告诉回收器void*指向的数据类型，以C++为例：

```c++
A *a = new A("cheng");
void *a_v = a;
delete ((A *)a_v);
```

#### 函数调用

##### 相关函数

```c
void func_void()               
{
    printf("func_void!\n");
}

void func_int(int value)       // 带参数的函数 
{
    printf("func_int par is: %d\n", value);
}

int func_int_return(int value) // 带返回值和参数的函数
{
    printf("func_int_return par is: %d\n", value);
    return value + 1024;
}
```



##### 空指针

```c
void *empty_func;
empty_func = func_void;                  // 空指针指向函数
(*(unsigned int (*)(void))empty_func)(); // 运行该函数：unsigned int(*)(void) 将空指针强制转换成函数指针
```



##### 空函数

```c
void (*func_p)(void);
func_p = func_void;
(*func_p)(); // 执行函数
```



##### 有参数的函数调用

```c
void (*func_int_p)(int);
func_int_p = func_int;
(*func_int_p)(100); // 执行函数
```



##### 有参数有返回值的函数调用

```c
int (*func_int_return_p)(int);
func_int_return_p = func_int_return;
int ans = (*func_int_return_p)(1024);

```



##### 回调函数实例

```c
typedef void (*callback)(void);

void my_call()
{
    printf("计算完毕!\n");
}

void mutmal(int a, int b, callback _call)
{
    int sum = a + b;
    printf("%d\n", sum);
    (*_call)();
}
mutmal(1024, 2048, my_call);
```

