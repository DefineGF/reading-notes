##### Person 类
```cpp
#ifndef PERSON_H_
#define PERSON_H_
#include <iostream>
#include <ostream>

namespace PersonValue {
class Person {
private:
    int age;
public:
    Person(int age) {
        std::cout << "Person(int): " << this << std::endl;
        this->age = age;
    }

    Person(const Person& p) {
        this->age = p.age;
        std::cout << "Person(const Person&): " << this << std::endl;
    }

    Person& operator=(const Person& p) {
        this->age = p.age;
        std::cout << "operator=: " << this << std::endl;
        return *this;
    }

    Person(Person&& p) {
        this->age = p.age;
        std::cout << "Person(Person&&): " << this << std::endl;
    }

    friend std::ostream& operator<<(std::ostream& os, const Person& p) {
        os << "age = " << p.age << "\n";
        return os;
    }

    ~Person() {
        std::cout << "~Person()" << std::endl;
    }

    void log() const {
        std::cout << "log: " << this << ", age = " << age << std::endl;
    }
}; // end class
}
#endif
```

##### emplace_back
```cpp
vector<Person> ps;
ps.emplace_back(Person(1));
```
输出结果：
> Person(int): 0x7fffd1ee22ec
Person(Person&&): 0x7fffca204e80
~Person()
~Person()

此时调用移动构造函数！


##### push_back
```cpp
Person left_person(1); // 左值
vector<Person> ps;
ps.push_back(left_person); // 拷贝构造函数


Person left_person(1);
vector<Person> ps;
ps.emplace_back(left_person); // 还是拷贝构造函数

// 使用move将左值 -> 右值，调用 移动构造函数
ps.emplace_back(std::move(left_person));
```

##### 数组
使用数组初始化vector，错误示范：
```cpp
int arr[] = {1, 2, 3};
vector<int> vi = arr; // error
vector<int> vi(arr);  // error
```
正确用法：
```cpp
vector<int> vi({1, 2, 3}); // or：vector<int> vi = {1, 2, 3}; 
```

针对复合数据类型：
```cpp
vector<Person> ps = {Person(1), Person(2), Person(3)};
```
1. 首先，创建中间数组会调用三次Person的构造函数；
2. 然后调用vector<Person> 构造函数时，又会调用三次Person的拷贝构造函数；
3. 最终调用 3 * 2 次析构函数！

注：
> for (auto i : ps) {} 这里遍历时，也会调用Person的构造函数创建对象，
可以使用迭代器，也可以修改为 for (auto& i : ps)

拷贝赋值：
```cpp
vector<Person> ps = {Person(1), Person(2), Person(3)};
vector<Person> ps2 = ps;
```
这里 ps 复制给 ps2，会逐个调用Person的拷贝构造函数！

移动复制：
```cpp
vector<Person> ps = {Person(1), Person(2), Person(3)};
vector<Person> ps2 = std::move(ps);
```
移动语义，直接将Person数组的控制权交给ps2，甚至没有调用Person移动构造函数！！！
此时打印ps的信息为空，size 和 capacity 均为 0；
相当于ps 直接改名 ps2. 

