#### friend

提供了普通函数或者类成员函数访问另一个类的私有或者保护成员 的机制；

##### 友元函数

*   在类的任何区域声明；
*   在类的外部定义；
*   只是一个普通的函数，而非类成员函数;

```c++
class FriendDemo {
    private:
        double weight;
    
    public:
        FriendDemo(double w) : weight(w) {};
    
    friend double getWeight(FriendDemo &FriendDemo);
};

double getWeight(FriendDemo &friendDemo) {
    return friendDemo.weight;
}

// 调用
int main() {
    FriendDemo friendDemo(73.0);
    cout << getWeight(friendDemo) << endl;
    return 0;
}
```



##### 友元类

*   在该类中声明，在类外实现

```c++
class MessageBox {
    private:
        const char *message;
    public:
        MessageBox(const char *msg) : message(msg) {};
    
    friend class Theft;
};
class Theft {
    public:
        const char *getMsgFromBox(MessageBox box) {
            return box.message;
        }
};

int main()
{
    MessageBox box("季子平安否");
    Theft theft;
    cout << theft.getMsgFromBox(box) << endl;
    return 0;
}
```

##### 优缺点

*   优点：提高程序的运行效率；
*   缺点：破坏了类的封装性和数据的透明性；



