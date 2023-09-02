#### inline

##### 内联函数定义

```c++
int add(int x, int y); // declaration
inline int add(int x, int y) { // definition
	return x + y;
}
```



##### 机制

1. 将 inline 函数体复制到 inline 函数调用点处；

2. 为所用 inline 函数中的局部变量分配内存空间；

3. 将 inline 函数的的输入参数和返回值映射到调用方法的局部变量空间中;



##### 注意事项

内联能提高函数效率，但并不是所有的函数都定义成内联函数！

内联是以代码膨胀(复制)为代价，仅仅**省去了函数调用的开销**，从而提高函数的执行效率。

