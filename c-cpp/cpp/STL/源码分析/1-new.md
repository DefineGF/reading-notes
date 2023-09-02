### 0-new实现

#### new operator

*   向堆申请一块内存；
*   执行构造函数；

#### operator new

间接调用malloc申请内存：

```cpp
void* __CRTDECL operator new(size_t const size)
{
    for (;;)
    {
        if (void* const block = malloc(size))
        {
            return block;
        }

        if (_callnewh(size) == 0)
        {
            if (size == SIZE_MAX)
            {
                __scrt_throw_std_bad_array_new_length();
            }
            else
            {
                __scrt_throw_std_bad_alloc();
            }
        }
    }
}
```

#### placement new

在已申请的内存上构建对象（包括栈），例子如下：

```cpp
int buf[10]; // stack
int* p = new(buf) int(1024); // buf 作为地址，1024表示初始值，
// 整个语句表示为：在buf所在位置（即数组第一个元素地址），设置int值为1024
```

#### 重载 new

```cpp
template <typename T> 
class Point {
public:
    Point(T d) : d(d) {
        cout << "constructor!" << endl;
    }

    void* operator new(size_t sz, void* p) {
        cout << "operator new" << endl;
        if (!p) {
            return ::operator new(sz);
        }
        return p;
    }

private:
    T d;
};

int main()
{
    char buf[sizeof(Point<int>)];
    Point<int>* ps = new (buf) Point<int>(1); // 在buf所在地址，构造一个 d = 1 的 Point
    return 0;
}
// 运行结果
// operator new 
// constructor!
```

#### 创建对象为例

##### 使用全局 operator new

```cpp
A *p = (A*)::operator new(sizeof(A)); // 调用全局的 operator new; (当然A也可以重载 operator new )
new(p) A();                           // 在申请的内存空间 p 上， 构造对象
p -> ~A(); 			                  // 析构
::operator delete(p);                 // 释放
```

##### 重写全局 operator new

```cpp
void* ::operator new(size_t size)
{
    std::cout<<"call global operator new"<<std::endl;
    return malloc(size);
}
```

##### 类中重写operator new

```cpp
#include <iostream>
class A
{
public:
    A()
    {
        std::cout<<"call A constructor"<<std::endl;
    }
 
    ~A()
    {
        std::cout<<"call A destructor"<<std::endl;
    }

    void* operator new(size_t size)
    {
        std::cout<<"call A::operator new"<<std::endl;
        return malloc(size);
    }
 
    void* operator new(size_t size, const std::nothrow_t& nothrow_value)
    {
        std::cout<<"call A::operator new nothrow"<<std::endl;
        return malloc(size);
    }
};

int main()
{
    A* p1 = new A;
    delete p1;
 
    A* p2 = new(std::nothrow) A; // std::nothrow: 申请内存失败不会抛出异常，而是返回nullptr
    delete p2;

    return 0;
}
```

