### 成员函数

#### const成员函数

```c++
char *getName() const; // 声明

char *Student::getName() const { // 实现
    return name;
}
```

- const 成员函数不允许调用非const 成员函数；
- const 成员函数不予许修改成员变量；

- const对象不能调用非const成员函数；

