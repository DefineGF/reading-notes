#### \_\_list\_node - 实现节点

```cpp
template <class T> struct __list_node 
{
  	typedef void* void_pointer;

	// 前后指针
  	void_pointer next;
  	void_pointer prev;
  	T data;// 属性
};
```

#### \_\_list\_iterator - 实现迭代器

```cpp
template<class T, class Ref, class Ptr> 
struct __list_iterator 
{
  	typedef __list_iterator<T, T&, T*>             iterator;	// 迭代器
  	typedef __list_iterator<T, const T&, const T*> const_iterator;
  	typedef __list_iterator<T, Ref, Ptr>           self;		
	
    // 迭代器是bidirectional_iterator_tag类型
  	typedef bidirectional_iterator_tag iterator_category;
  	typedef T value_type;
  	typedef Ptr pointer;
  	typedef Ref reference;
  	typedef size_t size_type;
  	typedef ptrdiff_t difference_type;
    ... 
};
```

#### list&#x20;

```cpp
template <class T, class Alloc = alloc>
class list 
{
protected:
    typedef void* void_pointer;
    typedef __list_node<T> list_node;	// 节点
    typedef simple_alloc<list_node, Alloc> list_node_allocator;	// 空间配置器
public:      
    // 定义嵌套类型
    typedef T value_type;
    typedef value_type* pointer;
    typedef const value_type* const_pointer;
    typedef value_type& reference;
    typedef const value_type& const_reference;
    typedef list_node* link_type;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    
protected:
    // 定义一个节点, 这里节点并不是一个指针.
    link_type node;
    
public:
    // 定义迭代器
    typedef __list_iterator<T, T&, T*>             iterator;
    typedef __list_iterator<T, const T&, const T*> const_iterator;
	...
};
```

