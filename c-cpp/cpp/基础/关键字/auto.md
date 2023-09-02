

#### auto

让编译器分析变量所属类型（auto变量必须有初始值）；

```c++
auto i = 0, *p = &i;    // 正确：i 为 int 类型；p为 整型指针
auto sz = 0, pi = 3.14; // 错误：sz 和 pi不是同一类型
```

##### 限制

*   不能在函数参数中使用；
*   不能用于类的非静态成员变量；
*   不能定义数组；
*   不能用于模板参数；

##### 应用

* 迭代器

* 泛型编程

  ```c++
  #include <iostream>
  using namespace std;
  
  class A{
  public:
      static int get(void){
          return 100;
      }
  };
  
  class B{
  public:
      static const char* get(void){
          return "http://c.biancheng.net/cplus/";
      }
  };
  
  template <typename T>
  void func(void){
      auto val = T::get();
      cout << val << endl;
  }
  
  func<A>();
  func<B>();
  
  ```

##### decltype

选择并返回操作数的数据类型；编译器并不实际调用函数f；

```c++
decltype(f()) sum = x; // sum的类型就是 f() 的返回值类型
```

变量：

```c++
const int ci = 0, &cj = ci;    // x 的类型是 const int
decltype(ci) x = 0;            // x 的类型是 const int 
decltype(cj) y = x;  		   // y 的类型是 const int &， 且绑定到 x 
decltype(cj) z; 			   // 错误：z是一个引用，必须初始化
```

引用：decltype 的表达式如果是加上括号的变量，结果是引用类型

```c++
int i = 12;
decltype((i)) d; // 错误：d 是int&, 必须初始化
decltype(i) e;   // 正确：e 是未初始化的int
```

