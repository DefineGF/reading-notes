

#### lambda

##### 委托

匿名委托：

```c#
Func<string, double, int> getLength = delegate (string content, double rate) {
	return (int)(content.Length * rate);
};
```

冗长lambda：

```c#
Func<string, double, int> getLength_1 = (string content, double rate) =>
{
	return (int)(content.Length * rate);
};
```

简洁lambda：

```c#
Func<string, double, int> getLength_2 = (content, rate) => (int)(content.Length * rate);
```

##### 列表

```c#
var files = new List<File>
{
	new File {Name = "E", Length = 120},
	new File {Name = "D", Length = 240},
	new File {Name = "F", Length = 360}
};
// 打印列表
Action<File> logFile = file => Console.WriteLine("file: name = {0}, length = {1}", file.Name, file.Length);
files.ForEach(logFile);
// 过滤
files.FindAll(file => file.Length > 250).ForEach(logFile);
// 排序
files.Sort((f1, f2) => f1.Length.CompareTo(f2.Length));
```



