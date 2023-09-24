##### 删除元素

```cpp
// vector
std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
// 使用迭代器遍历vector并删除符合条件的元素
for (auto it = numbers.begin(); it != numbers.end(); ) {
    if (*it % 2 == 0) {
        it = numbers.erase(it);  // 删除符合条件的元素，并返回指向下一个元素的迭代器
    } else {
        ++it;  // 继续下一个元素
    }
}

// map
std::map<int, std::string> data = {{1, "John"}, {2, "Alice"}, {3, "Bob"}, {4, "Jane"}};
// 使用迭代器遍历map并删除符合条件的键值对
for (auto it = data.begin(); it != data.end(); ) {
    if (it->second == "Alice") {
        it = data.erase(it);  // 删除符合条件的键值对，并返回指向下一个元素的迭代器
    } else {
        ++it;  // 继续下一个元素
    }
}
```

