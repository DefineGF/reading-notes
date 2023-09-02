### 对象

#### 创建对象

*   直接声明对象

    在栈上分配空间，栈上空间自动回收；

*   new 对象

    *   new 对象返回指针类型；
    *   在 heap 上分配空间，因此需要手动回收（delete），且要把指针设置为 NULL
    *   频繁调用场合不适合 new，需要申请内存和释放的过程；

##### 直接初始化

要求编译器使用普通的函数匹配来选择与我们提供的参数最匹配的构造函数；

```c++
string dots(10, ".");
string s(dots);
```

##### 拷贝初始化

要求编译器将右侧运算对象拷贝到正在创建的对象中（有时需要类型转换）；

通常使用拷贝构造函数完成；

```c++
string s2 = dots;
string null_book = "99-99";
string nines = string(100, '9');
```

不仅仅 ”=" 定义变量的时候会发生：

*   将一个对象作为实参传递给一个非引用的形参；

    ```c++
    void print(vector<int> _data) { ... }
    int main() {
    	vector<int> data;
    	print(data);
    }
    ```

*   将一个返回类型为非引用类型的函数返回一个对象：

    ```c++
    vector<int> getData() { return data; }  // error
    vector<int>& getData() { return data; }
    ```

*   用花括号列表初始化一个数组中的元素或一个聚合类中的成员：

    ```c++
    struct Data {
    	int val;
    	string s;
    };
    Data vall = {0, "cheng"};
    ```

*   初始化标准库容器或者调用insert() 或者 push()，容器会对元素进行拷贝初始化；（通常使用emplace成员创建元素直接初始化）

##### 隐式的类类型转换

```c++
class Person {
private:
    string name;
public: 
    Person() {}
    Person(int id) { name = to_string(id);	}
    Person(const string& _name) : name(_name) {}

    void logMsg() {
        cout << "name = " << name << endl;
    }
};

int main() {
    Person p_int = 18;					// 隐式转换
    Person p_str = string("cheng");		// 隐式转换
    p_int.logMsg();  // 18
    p_str.logMsg();  // cheng
    return 0;
}
```

禁止隐式转换explicit：

```c++
class Person {
... ...
	explicit Person(int id) { name = to_string(id);	}
    explicit Person(const string& _name) : name(_name) {}
... ...
};
Person p_int = 18;					// ERROR: 不存在从 "int" 转换到 "Person" 的适当构造函数
Person p_str = string("cheng");     // ERROR: 不存在从 "string" 转换到 "Person" 的适当构造函数
Person p_int(18);					// Right
Person p_str("cheng");				// R
```

