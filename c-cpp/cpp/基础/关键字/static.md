#### static

##### 函数中 静态变量

```c++
void demo() {
	static int count = 0; 
    count++; 
}

int main() {
    for (int i=0; i<5; i++)  
        demo(); 
}
// finally count = 5;
```

即使多次调用该函数，静态变量的空间也**只分配一次**;



##### 类中静态变量

- 类中静态变量由对象共享；
- 类的静变量不能通过构造函数初始化；



##### 类为静态变量

Dog.cpp 

```c++
#include <iostream>

class Dog
{
    public:
        Dog() {
            std::cout << "constructor!\n";
        }
        ~Dog() {
            std::cout << "deconstructor!\n";
        }
};
```

- 正常情况下：

    ```c++
    #include "Dog.cpp"
    #include <iostream>
    int main()
    {
        int i = 1;
        if (i) {
            Dog dog;
        }
        std::cout << "end of main!\n";
        system("pause");
        return 0;
    }
    ```

    输出结果为：

    > constructor!
    > deconstructor!
    > end of main!

    当结束 if 时，对象即被销毁；

- static情况下：

    > constructor!
    >
    > end of main!
    >
    > deconstructor!

    在main结束后调用析构函数, 这是因为静态对象的范围是贯穿程序的生命周期;



##### 类中静态函数

```c++
class Apple 
{
	public:
		logMsg() {
			std::cout << “this is an apple! " << std::endl;
		}
}

int main() {
	Apple::logMsg(); // static member function
	return 0;
}
```

