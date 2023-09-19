##### 基本操作

- 移除最低位1：x & (x - 1)
- 直接获取x二进制最低位的1：x & -x



#### 应用

##### 2的幂

```java
// way1
public boolean isPowerOfTwo(int n) {
    if (n <= 0) return false;
    return (n & (n - 1)) == 0;
} 
// way2
public boolean isPowerOfTwo(int n) {
    if (n <= 0) return false;
    return  (n & -n) == n;
} 
```

##### 4 的幂

```java
public boolean isPowerOfFour(int n) {
    if (n <= 0) return false;
    if ((n & (n - 1)) != 0) {
        return false; // 非 2 的幂
    }
    int index = 0;
    while (n > 0) {
        int a = n & 1;
        if (a == 1) {
            return index % 2 == 0;
        }
        n >>>= 1;
        index += 1;
    }
    return false;
}
```

