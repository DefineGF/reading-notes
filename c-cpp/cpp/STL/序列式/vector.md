#### c向量容器 vector \<T>

本质为长度可变的数组；

- 长度可变，内存不够则申请更多空间
- 尾部增加或者删除元素效率最高
- 其他地方插入删除元素时间复杂度：O(n)

##### 创建

```C++
#include <vector>
vector<double> values;
vector<int> primes {2,3,5,7};
vector<double> values(20); // 指定 vector 大小，默认初始值都为 0
vector<double> values<20, 1.0); // 指定元素初始值

// 使用 已有容器
vector<char> value1(5, 'c');
vector<char> value2(value1);

// 也可以指定 已有容器特定元素
int array[] = {1,2,3};
vector<int> value1(array, array + 2); // value1 保存 {1,2}；
vector<int> value2(std::begin(value1), std::begin(value1) + 1);
```



##### 增加容器容量 reserve()

更改 capacity， 而不修改 size

调用增加容量方法之后，之前创建好的迭代器都有可能失效；因为容器元素已经复制或者移动到了新的内存；



##### 添加元素 push_back() /  emplace_back()

```c++
vector<int> values{};
values.push_back(1);
values.emplace_back(2);
```

两者区别为：

- push_back():

    首先会创建这个元素，然后将这个元素 移动 或者 拷贝（之后销毁先前元素）进容器中

- emplce_back():

    直接在容器尾部创建这个元素；



##### 获取元素

- at[pos]
- reference front() 
- reference back()
- iterator begin()
- iterator end()



##### 插入元素 insert / emplace

- insert : 返回表示第一个新插入元素位置的迭代器。

    - iterator insert(pos, elem)：在pos指定位置 **之前** 插入新元素 elem

        ```c++
        vector<int> values {1,2};
        vector<int> :: iterator i;
        i = values.insert(values.begin() + 1, 3); // *i = 3
        ```

    - iterator insert(pos,n,elem): 在迭代器 pos 指定的位置之前插入**n 个元素 elem**

        ```c++
        demo.insert(demo.end(), 2, 5);//{1,3,2,5,5}
        ```

    - iterator insert(pos,first,last): 在迭代器 pos 指定的位置之前，插入其他容器（不仅限于vector）中位于 [first,last) 区域的所有元素

        ```c++
        array<int,3>test{ 7,8,9 };
        demo.insert(demo.end(), test.begin(), test.end());//{1,3,2,5,5,7,8,9}
        ```

    - iterator insert(pos,initlist): 在迭代器 pos 指定的位置之前，插入初始化列表（用大括号{}括起来的多个元素，中间有逗号隔开）中所有的元素

        ```
        demo.insert(demo.end(), { 10,11 });//{1,3,2,5,5,7,8,9,10,11}
        ```

        

- emplace(): 每次只能插入一个元素

    ```c++
    demo1.emplace(demo1.begin(), 3)
    ```

    emplace() 效率要 高于  insert() ; 后者先是创建元素，然后将元素移动/复制到容器中；

    前者则是直接在容器中创建元素。



##### 删除元素

- pop_back(): 删除最后一个元素，size  - 1
- erase(pos) : 删除 vector 容器中 pos 迭代器指定位置处的元素，并返回指向被删除元素下一个位置元素的迭代器。该容器的大小（size）会减 1，但容量（capacity）不会发生改变

- erase(begin, end): 刪除指定区域的所有元素；
- clear() 删除 vector 所有元素，size = 0，但是capacity不变



#### 小技巧

##### 倒置：

- 原地倒置：reverse(iterator1, iterator2);
- 新创建倒置：reverse_copy(iterator1, iterator2);





##### vector 添加 vector

```c++
vector<int> v1;
vector<int> v2;
v1.insert(v1.end(), v2.begin(), v2.end());
```



##### 复制问题

- 直接赋值：

    ```c++
    vector<int> data {1,2,3};
    vector<int> copy = data;
    copy[0] = 1024;
    cout << data[0] << endl;
    ```

- 源 vector：

    ```
    vector<int> v1 = {1,2,3};
    vector<int> v2(begin(v1) + 1, end(v1));
    ```

    