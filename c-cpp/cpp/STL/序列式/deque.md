#### 双端队列容器 deque\<T>

- 头部和尾部插入或者删除元素效率最高；
- 其他位置插入或者删除元素时间复杂度：O(n)
- 存储元素并不能保证所有元素都存储到连续内存空间中



##### 创建

```c++
deque<int> d; // 空
deque<int> d(10); // capacity = 10
deque<int> d(10, 5); // 默认初始值为 5
deque<int> d2(d1); 

int a[] = {1,2,3};
deque<int> d(arr.begin(), arr.end());
```



##### 常用方法

- begin / end

- resize() 修改实际元素个数

    ```c++
        deque<int> vals {1,2,3};
        cout << *vals.rbegin() << endl; // 最后一个元素，等同于 *(vals.end() - 1); 3
    
        vals.resize(16);
        cout << *vals.rbegin() << endl; // 0; size = 16
    ```

- shrink_to_fit() 将内存减少到等于当前元素实际所使用的大小

- front() / back(): 返回第一个/ 最后一个元素的引用

    - front() = 1；
    - end() = 3;

- push_back() / push_front() : 在序列的尾部 / 头部 添加一个元素

- pop_back() / pop_front(): 移除容器尾部 / 头部 的元素

- emplace_back() / emplace_front() ：尾部/ 头部添加元素

    



##### 底层分析

动态地以分段连续空间组合而成，随时可以增加一段新的空间链接起来；















