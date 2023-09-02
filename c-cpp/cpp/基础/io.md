### IO

#### 头文件

##### iostream

*   从流中读取数据：istream、wistream
*   向流中写入数据：ostream、wostream
*   读写流：iostream、wiostream

##### fstream

*   从文件读取数据：ifstream、wifstream
*   向文件中写数据：ofstream、wofstream
*   读写文件：fstream、wfstream

##### sstream

*   从string中读取数据：istringstream、wistringstream
*   向string中写入数据：ostringstream、wostringstream
*   读写string：stringstream、wstringstream

#### IO对象无拷贝或者赋值

```c++
ofstream out1, out2;
out1 = out2;           // error：不能对流对象赋值
```

