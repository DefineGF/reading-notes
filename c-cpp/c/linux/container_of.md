

##### 内容

```c
#define offsetof(TYPE, MEMBER) ((size_t) & ((TYPE *)0)->MEMBER)

#define container_of(ptr, type, member) (                  \
    {                                                      \
        const typeof(((type *)0)->member) *__mptr = (ptr); \
        (type *)((char *)__mptr - offsetof(type, member)); \
    })
```

参数：

- ptr：结构体内成员 member 的地址；
- member：结构体内成员；
- type：结构体类型

##### 作用

根据结构体某一成员的地址，获取整个结构体的地址。



##### 详情

步骤一：

```c
offsetof((size_t) & ((TYPE *)0)->MEMBER)
```

使用：

```c
struct student
{
    bool is_man;    // 1 字节
    int age;        // 4 字节
    double height;  // 8 字节
}; 				    // 16 字节
```

将 0 转换成 struct student类型，并获取成员地址：

```c
printf("%p\n", &((struct student *)0)->is_man);   // 0000000000000000
printf("%p\n", &((struct student *)0)->age);      // 0000000000000004
printf("%p\n", &((struct student *)0)->height);   // 0000000000000008
```

步骤二：

```c
#define container_of(ptr, type, member) (                  \
    {                                                      \
        const typeof(((type *)0)->member) *__mptr = (ptr); \
        (type *)((char *)__mptr - offsetof(type, member)); \
    })
```

> 注：第三行有类型检查的功能！
>
> 当传入的ptr并不是 type 中的 member 成员类型时，编译出错！

使用：

```c
struct student stu = {
    .is_man = true,
    .age = 22,
    .height = 172.0,
};
struct student *p_stu;
p_stu = container_of(&stu.age, struct student, age);
printf("age = %d\theight = %f\n", p_stu->age, p_stu->height);
```



