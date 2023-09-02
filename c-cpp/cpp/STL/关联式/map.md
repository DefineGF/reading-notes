 

### pair

##### 引入

```c++
#include <utility>
```

##### 构造方法

```c++
pair<string, double> p1; // "" 0
pair<string, string> p2("cheng", "dong");
pair<string, string> p3(p2);
pair<string, string> p4(make_pair("cheng", "dong")); // pair<string, string> p4 = make_pair("cheng", "dong");
pair<string, string> p5(string("cheng"), string("dong"));
```

##### 访问

p1.first

p1.second

当然可以直接赋值：p1.first = "cheng";



##### 运算符: < 、<= 、 > 、>=、 == 、!=

如果两者元素类型相同，那么比较的时候先 first 后 second



### map

#### 特征

- 各个元素的键必须是唯一的（不能重复），存入后也不能修改

- 根据元素键的大小，默认进行升序排序

    

#### 创建

##### 模板定义

```c++
template < class Key,                                     // 指定键（key）的类型
           class T,                                       // 指定值（value）的类型
           class Compare = less<Key>,                     // 指定排序规则 
           class Alloc = allocator<pair<const Key,T> >    // 指定分配器对象的类型
           > class map;
```

##### 生成对象

```c++
map<string, int> mMap; 
map<string, int> mMap {{"cheng", 22}, {"dong", 21}};
map<string, int> mMap {make_pair("cheng", 22), make_pair("dong", 21)};
```



#### 常用方法：

##### 判断存在：count(key)

返回键为 key 的键值对的个数，0 | 1；因此可用于判断是否含有当前键；

```c++
size_type count(const key_type& __x) const
{ return _M_t.find(__x) == _M_t.end() ? 0 : 1; }
```



##### 获取键值

```c++
mMap.at("cheng");
mMap["cheng"];
```

##### 搜索键之迭代器

```c++
iterator find(const key_type& __x){ 
	return _M_t.find(__x); 
}
```

注意：

如果存在则执行键的迭代器；如果不存在，指向 mMap.end()



##### 插入键

- **emplace**

```c++
// 两种都行
mMap.emplace("cheng", 22);
mMap.emplace(make_pair("yin", 23));
```

源码：

```c++
template<typename... _Args>
std::pair<iterator, bool> emplace(_Args&&... __args){ 
	return _M_t._M_emplace_unique(std::forward<_Args>(__args)...); 
}
```

注意返回的类型：

- 当map中已存在当前插入的键值对时，返回pair.first = iterator 指向map 中原键的 迭代器；bool值为 false；
- 若无要插入的键值对，那么插入之后返回插入迭代器，并 true；

```c++
mMap.emplace(make_pair("yin", 23));
mMap.emplace("wang", 24);
auto ans = mMap.emplace("wang", 25);
auto it = ++ans.first;
if (ans.second == false) {
	cout << "next value is: " << it -> first << endl;  // yin
}
```

- **insert**

```c++
mMap.insert(make_pair("wang", 25)); // 先创建 pair
```



##### 删除键

erase

```c++
// @return  The number of elements erased
size_type erase(const key_type& __x){ 
	return _M_t.erase(__x); 
}
```



##### 其他

- empty():  若容器为空，则返回 true；否则 false。

- size(): 返回当前 map 容器中存有键值对的个数。
- clear(): 清空 map 容器中所有的键值对，即使 map 容器的 size() 为 0



### multimap

与map相比，除了容器也会自行根据键的大小对存储的所有键值对做排序操作外，

和 map 容器的区别在于，multimap 容器中可以同时存储多（≥2）个键相同的键值对。



#### 创建

##### 模板

```c++
template < class Key,                                   // 指定键（key）的类型
           class T,                                     // 指定值（value）的类型
           class Compare = less<Key>,                   // 指定排序规则
           class Alloc = allocator<pair<const Key,T> >  // 指定分配器对象的类型
           > class multimap;
```



##### 常用方法

map 容器相比，multimap 未提供 at() 成员方法，也没有重载 [] 运算符；(因为键可有重复)

因此，不可以通过指定键获取指定指定键值对；

注意： count(key) 返回的值已经不限于 0 | 1 了

##### 新增方法

- lower_bound(key)

    返回一个指向当前 multimap 容器中第一个 **大于或等于**  key 的键值对的双向迭代器；

    如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。

- upper_bound(key)

    返回一个指向当前 multimap 容器中第一个大于 key 的键值对的迭代器。

    如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。

```c++
string my_key = "dong";
multimap<string, int> mMultimap {
        {"cheng", 22},
        {"dong", 21},
        {"yin", 23}
};
auto it1 = mMultimap.upper_bound(my_key); //  it1 -> first = "yin"
auto it2 = mMultimap.lower_bound(my_key); //  it2 -> first = "dong"
```

