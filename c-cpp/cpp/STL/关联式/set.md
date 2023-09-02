### set

会自动排序；

#### 创建

##### 模板

```c++
template < class T,                        // 键 key 和值 value 的类型
           class Compare = less<T>,        // 指定 set 容器内部的排序规则
           class Alloc = allocator<T>      // 指定分配器对象的类型
           > class set;
```

##### 创建

```c++
set<string> mSet;
set<string> mSet{"cheng", "dong"}
set<string> nSet(mSet);  // 拷贝构造函数

// 移动构造函数
set<string> createSet() {
	set<string> mSet {"cheng", "dong"};
	return mSet;
}
set<string> copySet(createSet());

set<string> copySet(mSet.begin(), mSet.end());
```

