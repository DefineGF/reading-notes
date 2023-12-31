#### 题目形式

- 计数（多少种方法)
- 求max / min
- 存在性

#### 解题步骤

- 确定状态：dp[i] 的含义
  - 最后一步
  - 子问题
- 转移方程
- 初始条件 和 边界条件



#### 例题

##### 凑硬币

> 三种硬币分别为2,5,7。且每种硬币足够多；
>
> 需要27元。使用最少的硬币组合刚好付清。

状态方程：

f(27) = min{ f(27 -  2) , f(27 - 5), f(27 -7) } + 1;



##### 01 背包问题

n件物体，不超过总容量 C 的情况下，装入背包的最大价值是多少？

状态方程：

```shell
f[i][c]: 
	i: 当前物品i;
	c: 当前容量;
转移方程：
f[i][c] = max( f[i - 1][c], f[i - 1][c - w[i]] + v[i])
表示截止当前物品，容量为c的背包，装下的最大价值 = max(不装入当前的，装入当前的)
```

代码如下：

```java
// v[i] 表示 i 商品价值
// w[i] 表示 i 商品体重
// C 背包容量
int knapsack(int[] v, int[] w, int C){
    int n = v.length; 
	int[][] f = new int[n + 1][C];
    
    for(int j = 1; j <= C; j++) {
        f[0][j] = (j >= w[0]) ?  v[0] : 0;
    }
       
    for(int i = 1; i < n; i++){       
        for(int j = 1; j <= C; j++){  
            if(j < w[i]){	
                // 当前容积尚不能容纳当前物品
                f[i][j] = f[i - 1][j];
            } else{					 
                // 可容纳当前物品时，可以选择不装入；也可以选择装入；
                // 装入时的价值 = 当前体积除去当前商品时的体积所能装下的最大价值 + 当前商品价值
                f[i][j] = max(f[i - 1][j], f[i - 1][j - w[i]] + v[i]);
            }
        }
    }
    return f[n - 1][C];
}
```

内存压缩：当前物品状态只与 上一物品状态相关；因此采用一维数组即可；

```java
for(int i = 0; i < n; i++){
    for(int j = C; j >= w[i]; j--)
        f[j] = max(f[j], f[j - w[i]] + v[i]);
}
return f[C];
```

> 第二次循环是自右向左，如果从左向右，那么就会覆盖掉原来的值；此后如果使用到之前 i - 1 的值，已经被 i 值覆盖了；



##### 完全背包

与01背包不同就是每种物品可以有无限多个