

##### 运行时获取对象类型

如果想通过基类的指针获得派生类的数据类型，基类必须带有虚函数；

小例子：

```cpp
#include <iostream>
#include <typeinfo>

using namespace std;


class Flyable                       // 能飞的
{
public:
    virtual void takeoff() = 0;     // 起飞
    virtual void land() = 0;        // 降落
};

class Bird : public Flyable         // 鸟
{
public:
    void foraging() { cout << "觅食" << endl; }           // 觅食
    virtual void takeoff() { cout << "鸟起飞" << endl; }
    virtual void land() { cout << "鸟降落" << endl; }
    virtual ~Bird(){}
};

class Plane : public Flyable        // 飞机
{
public:
    void carry() { cout << "飞机运输!" << endl; }              // 运输
    virtual void takeoff() { cout << "飞机起飞! " << endl; }
    virtual void land() { cout << "飞机降落!" << endl; }
};


void do_something(Flyable* f) {
    f->takeoff(); 

    cout << typeid(*f).name() << endl;

    if (typeid(*f) == typeid(Bird)) {
        Bird* b = dynamic_cast<Bird*>(f);
        b->foraging();
    } else if (typeid(*f) == typeid(Plane)) {
        Plane* p = dynamic_cast<Plane*>(f);
        p->carry();
    }
    f->land();
}

int main()
{
    Bird* b = new Bird();
    do_something(b);
    delete b;

    Plane* p = new Plane();
    do_something(p);
    delete p;
    
    return 0;
}
```

