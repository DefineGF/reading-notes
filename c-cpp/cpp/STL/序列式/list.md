#### list 双向链表

##### 创建

与其他类似

```c++
list<int> lt(4);
lt.push_back(1); // 此时 size = 5 (初始创建时有 4 个 元素为 0 的值)
```



##### 常用方法

- front() / back()

- assign() 新元素替换旧元素

    ```c++
    list_ont.assign(list_two.begin(), list_two.end());
    ```

    

- emplace_front() / emplace_back() / push_front() / push_back()

- pop_front() / pop_back()

- emplce() / insert()

- arase() / remove() 

- unique() 删除容器中相邻的重复元素

- merge()  合并另个有序的list

- sort() 排序

- reserve()  反转

    

