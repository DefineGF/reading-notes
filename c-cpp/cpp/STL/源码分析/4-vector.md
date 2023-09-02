#### 基本结构

```cpp
template <class T, class Alloc = alloc>
class vector 
{
public:
  	// 定义vector自身的嵌套型别
    typedef T value_type;
    typedef value_type* pointer;
    typedef const value_type* const_pointer;
    // 定义迭代器, 这里就只是一个普通的指针
    typedef value_type* iterator;
    typedef const value_type* const_iterator;
    typedef value_type& reference;
    typedef const value_type& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    ...
  protected:
    typedef simple_alloc<value_type, Alloc> data_allocator;	// 设置其空间配置器
    iterator start;				// 使用空间的头
    iterator finish;			// 使用空间的尾
    iterator end_of_storage;	// 可用空间的尾 (因为容量大于实际元素占用空间)
    ...
};
```

#### &#x20;构造函数

```cpp
vector() : start(0), finish(0), end_of_storage(0) {}		// 默认构造函数
explicit vector(size_type n) { fill_initialize(n, T()); } 	// 必须显示的调用这个构造函数, 接受一个值
vector(size_type n, const T& value) { fill_initialize(n, value); } // 接受一个大小和初始化值. int和long都执行相同的函数初始化
vector(int n, const T& value) { fill_initialize(n, value); }	
vector(long n, const T& value) { fill_initialize(n, value); }
vector(const vector<T, Alloc>& x); 	// 接受一个vector参数的构造函数
```

#### &#x20;析构函数

```cpp
void deallocate() 
{
	if (start) 
        data_allocator::deallocate(start, end_of_storage - start);
}
// 调用析构函数并释放内存
~vector() 
{ 
	destroy(start, finish);
	deallocate();
}
```

#### 属性

```cpp
public:
	// 获取数据的开始以及结束位置的指针. 记住这里返回的是迭代器, 也就是vector迭代器就是该类型的指针.
    iterator begin() { return start; }
    iterator end() { return finish; }
	// 获取值
	reference front() { return *begin(); }
    reference back() { return *(end() - 1); } 
	// 获取右值
	const_iterator begin() const { return start; }
    const_iterator end() const { return finish; }
	const_reference front() const { return *begin(); }
	const_reference back() const { return *(end() - 1); }
    // 获取基本数组信息
	size_type size() const { return size_type(end() - begin()); }	 // 数组元素的个数
    size_type max_size() const { return size_type(-1) / sizeof(T); }	// 最大能存储的元素个数
    size_type capacity() const { return size_type(end_of_storage - begin()); }	// 数组的实际大小
```

#### 方法

判空：

```cpp
bool empty() const { return begin() == end(); }	
```

清除：

```cpp
v.earse(iterator);
v.earse(iterator1, iterator2); // 左闭右开

// 全部
void clear() { earse(begin(), end()); }
```

重载：

```cpp
reference operator[](size_type n) { return *(begin() + n); }
const_reference operator[](size_type n) const { return *(begin() + n); }

// 当vector<int> data = {1, 2, 3};
data.end()[-1] = 3; // 不常用
```

插入：

*   如果数组有备用空间，先移动元素，再插入数据；
*   如果没有空间，重新申请原始空间 \* 2 + 1 的空间，再拷贝同时插入；

遍历：

```cpp
for_each(v.begin(), v.end(), [](int a) { std::cout << "a " <<  a << std::endl; }); // #include <algorithm>
```

#### 其他

*   在生命周期结束后会自动调用析构函数, 用户不再手动的释放内存, 也不会出现内存泄露的问题, 用户也可以主动调用析构函数释放内存：v.\~vector()

