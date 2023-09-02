#### 函数指针

```cpp
int add(int a, int b) {
    return a + b;
}
```



##### 形式1

```cpp
int(*func)(int, int);
func = add;
cout << func(10, 12) << endl;
```



##### 形式2

```cpp
typedef int(*p_func)(int, int);

// 需要声明一下
p_func f = add;
cout << f(10, 22) << endl;
```

##### 数组形式



```cpp
// 形式1
void(*fun[3])(void*);

// 形式2
typedef void(*pfun)(void*);
pfun fun[3];  

// 然后赋值
fun[0] = fun1;  
fun[1] = fun2;  
fun[2] = fun3; 
```



#### lambda



##### 完整例子

```c++
auto f = [](int a) -> int {
	return a + 1;
}
cout << f(1) << endl;
```

##### 捕获列表

*   \[] 不捕获任何对象
*   \[&] 捕获外部作用域所有变量 （按引用捕获）
*   \[=] 捕获外部作用域所有变量 （按值捕获）
*   \[=, \&v] 捕获外部作用域所有变量 按引用捕获v 变量
*   \[bar] 按值捕获bar 变量
*   \[this] 获取当前类中的this指针

##### 实例

```c++
void lambda_test() {
	std::vector<int> v = { 1,2,3,4,5,6 };
	int event_count = 0;
	for_each(v.begin(), v.end(), [&event_count](int val)
		{
			if (val % 2 == 0) {
				++event_count;
			}
		});
	cout << "get the even_count is: " << event_count << endl;
}
```

