#### const 作用

- 防止修改，起保护作用

  ```c++
  void func(const int i) {
  	i++; // error
  }
  ```

- 相对于#define 来讲节省空间

  const定义常量从汇编的角度来看，只是给出了对应的**内存地址**，而不是像#define一样给出的是立即数；

  所以，const定义的常量在程序运行过程中只有一份拷贝，而#define定义的常量在内存中有若干个拷贝。



#### 使用

##### 不同文件使用

```c++
// file1.cpp
extern const int ext = 16;

// file2.cpp
#include "file1.cpp"
#include <iostream>
extern const int ext;
int main()
{
	std::cout << "ext = " << ext << std::endl; // 16
	return 0;
}
```



##### const 与 指针

```c++
const int *p; // 指针指向的内容不可修改
int* const p; // 指针不可指向其他位置
```



##### const 与 函数

```c++
const int* func(); // 指针指向的内容不能更改
int* const func(); // 指针不可更改
```



##### const 与 类

- 在一个类中，任何不会修改数据成员的函数都应该声明为const类型；
- const**成员函数**不慎修改数据成员，或者调用了其它非const成员函数，则出错；
- const对象只能访问const成员函数, 而非const对象可以访问任意的成员函数,包括const成员函数；

```c++
/*
 * 首个 const 用来修饰函数的返回值 表示返回值是 const 类型；
 * 尾部 const 用来表示是 常成员函数，只能读取成员变量的值，而不能修改成员变量的值；
*/
const Screen& display(std::ostream& os) const{
	do_display(os);
	return *this;
}
```

