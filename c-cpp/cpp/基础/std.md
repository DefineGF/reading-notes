##### accumulate

```cpp
#include <numeric>
#include <functional>
// 求和
std::vector<int> vec = {2, 3, 4};
int sum = std::accumulate(vec.begin(), vec.end(), 1); // 1 作为初始值

// 求积
int multi = std::accumulate(vec.begin(), vec.end(), 1, std::multiplies<int>());
```

##### multiplies

```cpp
#include <functional> // std::multiplies
#include <algoritm> // std::transform
```

##### remove

原函数：

```cpp
template <class ForwardIterator, class T>
ForwardIterator remove (ForwardIterator first, ForwardIterator last, const T& val)
{
  ForwardIterator result = first;
  while (first!=last) {
    if (!(*first == val)) {
      if (result!=first)
        *result = move(*first);
      ++result;
    }
    ++first;
  }
  return result;
}
```

以 {1, 2, 3, 1, 2, 3}，删除 3 为例：

1.  从前向后遍历
    1.  遇到1,2 first和result均向后；
    2.  result先指向3，然后firs跳过3指向下一元素1；
    3.  将first指向元素1复制到result 3 位置；
2.  最后 -> {1, 2, 1, 2, 2, 3}

