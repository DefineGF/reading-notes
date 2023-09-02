##### 2.2 运算符优先级

判断某位二进制为1：

```c
if (0x01 & num)      // 没问题
if (0x01 & num != 0) // 错误：!= 运算符优先级高于 &
```



其他：

```c
r = hi << 4 + low; // + 优先级 移位
// 修改1
r = (hi << 4) + low;

// 修改2
r = hi << 4 | low;
```



 

##### 3.1 指针与数组

```c
int calendar[12][31];
int (*monthp)[31];    // ap 是一个指向（拥有31个整形元素的数组）的指针
monthp = calendar;
```



##### 3.5 空指针 与 空字符串

```c
#define NULL 0
if (p == (char*)0) // 没问题

if (strcmp(p, (char*)0) == 0) {} // 非法：strcmp 会查看指针参数指向内存中内容 
```

