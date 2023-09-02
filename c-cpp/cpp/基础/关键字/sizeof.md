#### 类大小计算

##### 空类大小 = 1 字节

```c++
class A{};
A a;
sizeof(a);  // 1
```



##### 成员变量 与 类存储空间

- 虚函数本身、**成员函数**（包括静态与非静态）和 **静态数据成员**都是不占用类对象的存储空间；

- 普通成员变量需要考虑内存对齐；

    ```c++
    class A {
        public:
            int age = 18;
    };
    sizeof(a);  // 4
    
    class A {
        public:
            double weight = 75.0;
    };
    sizeof(a);  // 8 
    
    class A {
        public:
            int age = 18;
            double weight = 75.0;
    };
    sizeof(a); // 16
    
    class A {
        public:
            int age = 18;
            double weight = 75.0;
            const char* name = "cheng jian ming\0";
    
    };
    sizeof(a); // 24 (64b os 指针占用 8B)
    ```



##### 多个虚函数

对于包含虚函数的类，不管有多少个虚函数，只有一个虚指针vptr的大小;



##### 普通继承

派生类继承了所有基类的函数与成员，要按照字节对齐来计算大小；



##### 虚函数继承

虚函数继承，不管是单继承还是多继承，都是继承了基类的vptr。(32位操作系统4字节，64位操作系统 8字节)！