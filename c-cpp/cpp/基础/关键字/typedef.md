#### typedef

##### C语言常见用法

```c
typedef int bool;
#define 0 False
#define 1 True

void func() {
    bool is_man = False;
}
```



##### 进阶

void (*Fun)(void)：定义了一个变量Fun，这个Fun变量是一个指针，指向返回值和参数都是空的函数的指针；

typedef void(*Func)(void)：声明一个 `void(*)()`类型的函数指针Func；



- 原始

    ```c++
    void log() {
        cout << "log the message" << endl;
    }
    
    int main() {
        void (*func)();
        func = &log;
        return 0;
    }
    ```

- 修改

    ```c++
    typedef void (*func)();
    
    void log() {
        cout << "log the message" << endl;
    }
    
    int main() {
        func f;
        f = &log;
        f();
        return 0;
    }
    ```



