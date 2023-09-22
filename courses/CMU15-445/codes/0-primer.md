#### 环境准备

##### 安装 clang12





##### 编译项目

```shell
# 设置CMakeLists.txt 文件并不起作用
# 指定cmake 命令方才有效
cmake -DCMAKE_C_COMPILER=/usr/bin/clang-12 -DCMAKE_CXX_COMPILER=/usr/bin/clang++-12 ..
```



#### Matrix

##### 部分源码

```cpp
template <typename T>
class Matrix {
 protected:
  Matrix(int rows, int cols) : rows_(rows), cols_(cols) {
    int len = rows * cols;
    linear_ = new T[len];
    memset(linear_, 0, sizeof(T) * len);
  }

  int rows_;
  int cols_;
  T *linear_;

 public:
  virtual auto GetRowCount() const -> int = 0;
  virtual auto GetColumnCount() const -> int = 0;
  virtual auto GetElement(int i, int j) const -> T = 0;
  virtual void SetElement(int i, int j, T val) = 0;
  virtual void FillFrom(const std::vector<T> &source) = 0;
    
  virtual ~Matrix() { delete[] linear_; }
};
```



#### RowMatrix

##### 部分源码

```cpp
template <typename T>
class RowMatrix : public Matrix<T> {
public:
  RowMatrix(int rows, int cols) : Matrix<T>(rows, cols) {
    data_ = new T *[rows];
    for (int i = 0; i < rows; i++) {
      data_[i] = this->linear_ + i * cols;
    }
  }
  ~RowMatrix() override { delete[] data_; }
private:
  T** data_; 
}
```

比较有意思的是：

- 父类Matrix 持有矩阵的所有数据；
- RowMatrix 只是通过 data_ 指向 父类Matrix 中的数据罢了；



