实例

```c++
auto get_student(int id) { // 被推断为 std::tuple<double, char, std::string)
	if (id == 0) {
		return std::make_tuple(3.8, 'M', "cheng");
	}
    return std::make_tuple(0.0, 'D', "null"); // 如果只写 0， 编译错误 推断失败
} 
auto student = get_student(0);
cout << std::get<0>(student);

double gpa;
char sex;
std::string name;
// 拆包
std::tie(gpa, sex, name) = get_student(0);
```

