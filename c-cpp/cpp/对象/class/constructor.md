#### 构造函数

##### 默认构造函数

*   编译器只有发现类没有构造函数的时候，才会生成一个默认构造函数；一旦定义了其他构造函数，除非再定义一个默认构造函数，否则该类没有构造函数；
*   默认构造函数可能执行错误的操作：内置类型（int，double等）或复合类型（struct，class，指针，数组，enum等等）默认初始化时是未定义的；
*   有时候编译器不能为某些类构造默认的构造函数，比如类中包含一个其他类型的成员且这个成员类型没有默认构造函数；

如果成员是 const、引用、或者属于某种未提供默认构造函数的类类型，须通过构造函数初始值列表为这些成员提供初值；

##### 缺少默认构造函数

```cpp
class NoDefault {
public:
    NoDefault(const string&);
};

struct A {
    NoDefault a_mem;
};

A a;            // error：不能为A合成构造函数
struct B {
    B() { }     // error：提示 NoDefault 没有默认构造函数
    NoDefault b_mem;
};
```

##### 构造方法调用

```cpp
class Person {
private:
    int age;

public:
    Person() { 
        age = 0; 
        std::cout << "run the defacult constructor!" << std::endl;
    }
    Person(int a) {
        age = a;
        std::cout << "run the Person(int a) constructor!" << std::endl;
    }

    Person(const Person& person) {
        age = person.age;
        std::cout << "run the copy constructor!" << std::endl;
    };

    void operator=(const Person& person) {
        age = person.age;
        std::cout << "run the operator=" << std::endl;
    }
};
```

执行：

```c++
Person p1(18);		// run the Person(int a) constructor!
Person p2 = p1;	    // run the copy constructor!                  (注意 & 注意 & 注意)

Person p3;			// run the defacult constructor!	(此时p3创建)
p3 = p1;            // run the operator= 						  (注意 & 注意 & 注意)   与第2
```

##### 特殊

```cpp
class Person{
private:
	int *age;
public:
	Person(int* _age) { age = _age; }
}

// 调用
Person p = new int(4); // 等价于 Person p(new int(4)); 和上面2 & 4 行
```

##### 拷贝构造函数

```c++
class Integer {
private:
    string name_;
    int* ptr_;

public:
    // 参数为常量左值引用的深拷贝构造函数，不改变 source.ptr_ 的值
    Integer(const Integer& source) : ptr_(new int(*source.ptr_)) {
        cout << "Call Integer(const Integer& source)" << endl;
    }
    
    // 参数为左值引用的深拷贝构造函数，转移堆内存资源所有权，改变 source.ptr_ 的值
    Integer(Integer& source) : ptr_(source.ptr_) {
        source.ptr_ = nullptr;
        cout << "Call Integer(Integer& source)" << endl;
    }
    
    Integer(int value) : ptr_(new int(value)) {
        cout << "Call Integer(int value)" << endl;
    }

    ~Integer() {
        cout << "Call ~Integer()" << endl;
        delete ptr_;
    }

    int GetValue(void) { return *ptr_; }
};
```

##### default 和  delete

*   MyClass() = delete;  表示删除默认构造函数
*   MyClass() = default; 表示默认存在构造函数

当类中含有不能默认拷贝成员变量时，可以禁止默认构造函数的生成：

*   MyClass(const MyClass&) = delete; 								表示删除默认拷贝构造函数，即不能进行默认拷贝；
*   MyClass& operator= (const MyClass&) = delete;           表示删除默认拷贝构造函数，即不能进行默认拷贝；

#### 实例分析

##### 实例一

```c++
class Person
{
public:
    Person() : age(new int(0))
    {
        std::cout << "Person();         this_address: " << this << "; age_addr: " << age << std::endl;
    }
    Person(const Person &p) : age(new int(*p.age))
    {
        std::cout << "Person(Person);   this_address: " << this << "; age_addr: " << age << std::endl;
    }
    ~Person()
    {
        std::cout << "~Person();        this_address: " << this << "; age_addr: " << age << std::endl;
        delete (age);
    }
private:
    int *age;
};
```

调用：

```c++
Person get_person()
{
    return Person();
}
int main()
{
    Person p = get_person();
    return 0;
}
```

取消编译优化：

```shell
g++ rvalue_1.cpp -fno-elide-constructors -o rvalue1
```

运行结果：

```shell
Person();         this_address: 0x61fdb8; age_addr: 0xe25f40  // Person()
Person(Person);   this_address: 0x61fe08; age_addr: 0xe25f60  // 函数返回值
~Person();        this_address: 0x61fdb8; age_addr: 0xe25f40  // 函数结束，局部变量析构

Person(Person);   this_address: 0x61fe00; age_addr: 0xe25f40  // Person p = get_person() 拷贝构造函数
~Person();        this_address: 0x61fe08; age_addr: 0xe25f60  // 右值析构
~Person();        this_address: 0x61fe00; age_addr: 0xe25f40  // 程序结束
```

添加移动拷贝构造函数：

```c++
Person(Person &&p) : age(p.age)
{
    p.age = nullptr;
    std::cout << "move constructor; this_address: " << this << "; age_addr: " << age << std::endl;
}
```

运行结果：

    Person();         this_address: 0x61fdb8; age_addr: 0x1b5f40
    move constructor; this_address: 0x61fe08; age_addr: 0x1b5f40
    ~Person();        this_address: 0x61fdb8; age_addr: 0

    move constructor; this_address: 0x61fe00; age_addr: 0x1b5f40
    ~Person();        this_address: 0x61fe08; age_addr: 0
    ~Person();        this_address: 0x61fe00; age_addr: 0x1b5f40

> 当类中同时包含拷贝构造函数和移动构造函数时，如果使用临时对象初始化当前类的对象，编译器会**优先调用移动构造函数**来完成此操作。只有当类中没有合适的移动构造函数时，编译器才会退而求其次，调用拷贝构造函数

继续测试：

```c++
Person p1;
Person p2 = p1;			   // 其中, p1 属于左值，调用 拷贝构造函数
Person p3 = std::move(p1); // move 将p1 转换为右值，调用 移动构造函数
```

运行结果：

```shell
Person();         this_address: 0x61fe08; age_addr: 0x1095f40
Person(Person);   this_address: 0x61fe00; age_addr: 0x1095f60
move constructor; this_address: 0x61fdf8; age_addr: 0x1095f40

~Person();        this_address: 0x61fdf8; age_addr: 0x1095f40
~Person();        this_address: 0x61fe00; age_addr: 0x1095f60
~Person();        this_address: 0x61fe08; age_addr: 0
```

冷静下来分析上一次测试，Person p = get\_person(); 函数返回值做 右值 处理，因此直接调用 移动构造函数\~
