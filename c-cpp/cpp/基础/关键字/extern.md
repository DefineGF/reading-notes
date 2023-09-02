#### extern

提供一个全局变量的引用，全局变量对所有的程序文件都是可见的，可以这么理解，extern 是用来在另一个文件中声明一个全局变量或函数。extern 修饰符通常用于当有两个或多个文件共享相同的全局变量或函数的时候。



##### 实例一

config.h

```c
int SIZE = 16;    // 这里给 全局变量 赋初值
```

A.c

```c
#include "config.h"
#include <stdio.h>
void a_log()
{
    printf("get variable: %d\n", SIZE);
}
```

B.c

```c
#include "config.h"
#include <stdio.h>
void set_size(int new_size)
{
    SIZE = new_size;
    printf("b get variable: %d\n", SIZE);
}
```

Test.c

```c
#include <stdio.h>
extern void a_log();
extern void set_size(int new_size);
int main()
{
    a_log();
    set_size(1024);
    a_log();
    return 0;
}
```

编译结果：

```shell
gcc A.c B.c Test.c -o test
o:B.c:(.data+0x0): multiple definition of 'SIZE'
o:A.c:(.data+0x0): first defined here
```

原因：

如果在.h文件中定义全局变量并赋初值，那么多个引用此头文件的C文件中同样存在相同变量名的拷贝；

由于该变量赋了初值，所以编译器将会将此变量放入 Data 段；

但是在链接阶段，在Data段中存在多个相同的变量，他无法将这些变量统一成变量。

修改：

**取消全局变量赋初值**，编译器会将之放入 Bss段，链接器会对Bss段的多个同名变量仅分配一个存储空间。