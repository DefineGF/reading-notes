##### 使用形式

```cpp
// 实例1
typedef unique_ptr<unordered_map<string, int>> UPtrMapSI;

using UPtrMapSI = unique_ptr<unordered_map<string, int>>; 

// 实例2
typedef void (*FP)(int);
using FP = void(*)(int);
```



#### 模板

##### 别名模板

```cpp
template <typename T>
using mVec = vector<T>;
// 使用
mVec<int> vec;
```

typedef 实现：

```cpp
template<typename T>
struct mVec {
    typedef vector<T> type;
}
mVec<int>::type vec; // 使用
```



##### 复杂用法：在别的模板类中使用

```cpp
template<typename T>
class Person {
private:
    typename mVec<T>::type vec; // 需要添加 typename 
    							// 如果using，那么：mVec<T> vec
    							// 因为 mVec<T>::type 可能是一个成员变量，而 mVec<int> 为一个模板类型
};
```

