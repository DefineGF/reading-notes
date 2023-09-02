#### 数组容器 array<T, N>

 存储 N 个 T 类型的元素，一旦建立则长度不变

```c++
namespace std{
    template <typename T, size_t N>
    class array;
}

#include <array>
using namespce std;
// array<double, 10> values; 未指定数组元素值，初始值不定
array<double, 10> values {0.5,1.0,1.5,,2.0};  // 其余四个元素都被初始化为 0.0
```



##### 迭代器

- begin() / end() :

    - begin():

        返回指向容器中第一个元素的正向迭代器；如果是 const 类型容器，在该函数返回的是常量正向迭代器。cbegin()和 begin() 功能类似，只不过其返回的迭代器类型为常量正向迭代器，**不能用于修改元素**。

    - end():

        返回指向容器最后一个元素之后一个位置的正向迭代器；如果是 const 类型容器，在该函数返回的是常量正向迭代器。

        cend() 和 end() 功能相同，只不过其返回的迭代器类型为常量正向迭代器，不能用于修改元素。

        

- rbegin() / rend()

    **反向迭代器使用 ++ 获得上一个元素引用** 

    - rbegin() :返回指向最后一个元素的反向迭代器；如果是 const 类型容器，在该函数返回的是常量反向迭代器。
    - rend(): 返回指向第一个元素之前一个位置的反向迭代器。

![迭代器的具体功能示意图](http://c.biancheng.net/uploads/allimg/191128/2-19112Q14QE40.gif)



##### 遍历实例：

```C++
// at(n) 返回容器中n位置处的引用，自动检查范围，访问更安全    
array<int, 4> values;
    for (int i = 0; i < values.size(); i++) {
        values[i] = i * 10;
        // values.at(i) = i * 10;
    }

    if (!values.empty()) {
        // auto使编译器自动判定变量类型
        for (auto val = values.begin(); val < values.end(); val++) {
            cout << *val << endl;
        }
    }
    int sum = 0;
    for (auto val : values) {
        sum += val;
    }
```

