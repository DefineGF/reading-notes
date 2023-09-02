被**mutable**修饰的变量，将永远处于可变的状态，即使在一个const函数中。

实例

```cpp
class Person {
private:
	int age;
	int invoke_count; // 统计函数调用次数
public:
	int getAge() const;
};
```

如果要统计 getAge() 函数调用的次数，那么需要在 getAge() 函数中 ++invoke*count；用于const的限制，因此需要将invoke\_count 修改为:*

```cpp
mutable int invoke_count; 
```

