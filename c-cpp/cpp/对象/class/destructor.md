#### 析构函数

析构函数并不会破坏对象本身，只是做些清理工作（比如回收动态分配的空间）；对象本身回收由系统进行

 - 全局对象存在程序的整个生命周期

 - 局部动态对象退出该对象的作用域时执行析构函数；

 - 局部静态对象存在于创建后的整个程序生命周期，程序结束时在全局对象的析构函数执行前执行；

 - 动态创建的对象（new）必须动态撤销，即在执行delete时候执行析构函数；

```c++
Object one(1);
int main()
{
    Object *ptr;
    Object two(2);
    {
        Object three(3);
        ptr = new Object(4);
        static Object five(5);
    }
    delete ptr;
    return 0;
}
```

​	上述例子中，执行析构函数的顺序是：3 (局部动态) -> 4 (动态delete) -> 2 (局部动态)  -> 5 (局部静态) -> 1 (全局对象)



- **生命周期**：对象**生命周期结束**，会调用析构函数。

- **delete**：调用delete，会删除指针类对象。

- **包含关系**：对象Dog是对象Person的成员，Person的析构函数被调用时，对象Dog的析构函数也被调用。

- **继承关系**：当Person是Student的父类，调用Student的析构函数，会调用Person的析构函数

##### 实例一

```c++
class Base {
public:
    Base() {
        cout << "Base -> constructor\n";
    }
    ~Base() {
        cout << "Base -> destructor\n";
    }

    void log() {
        cout << "Base: log()" << endl;
    }
};

class Derived : public Base {
public:
    int new_var = 16;
    Derived() {
        cout << "Derived -> constructor\n";
    }

    ~Derived() {
        cout << "Derived -> destructor\n";
    }

    void log() {
        cout << "Derived: log()" << endl;
    }
};
```

- 调用一

    ```c++
    Base* ob = new Derived();
    delete ob;
    ```

    输出结果：

    > Base -> constructor
    > Derived -> constructor
    > Base -> destructor

    小细节：

    ```c++
    sizeof(*ob); // 1; 空对象 sizeof(ob) = 8B 指针大小
    ob -> log(); // Base: log()
    ```

    

- 调用二

    ```c++
    Derived* de = new Derived();
    delete de;
    ```

    输出结果：

    > Base -> constructor
    > Derived -> constructor
    > **Derived -> destructor**
    > Base -> destructor

    小细节：

    ```c++
    sizof(*de); // 4; 增加了 int 变量
    de -> log(); // Derived: log()
    ```


##### 实例二

将基类析构函数设置为 virtual，其余不变：

```c++
class Base {
public:
    Base() {
        cout << "Base -> constructor\n";
    }
    
    virtual ~Base() {
        cout << "Base -> destructor\n";
    }

    void log() {
        cout << "Base: log()" << endl;
    }
};
```

- 调用一：

    ```c++
    Base* ob = new Derived();
    ob -> log();
    delete ob;
    ```

    - 输出结果：

        > Base -> constructor
        > Derived -> constructor
        > **Base: log()**
        > Derived -> destructor
        > Base -> destructor

        相比之下，当Base中的析构函数为virtual时候，Base实例对象则会调用子类的析构函数；

        

    - 小细节

        > sizeof(*ob);        // 8;	增加虚函数，即vptr大小

- 调用二

    ```c++
     Derived* de = new Derived();
     de -> log();
     delete de;
    ```

    - 输出结果：

        > Base -> constructor
        > Derived -> constructor
        > Derived: log()
        > Derived -> destructor
        > Base -> destructor

    - 小细节:

        ```c++
        sizeof(*de); // 16; 相对于Base不仅增加了 int， 同时也继承了 Base 中的 vptr
        ```

        

    
