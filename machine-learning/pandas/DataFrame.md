#### 相关系数 - corr 

DataFrame.corr(method='method\_name', min\_periods)

method\_name:

*   pearson: 针对线性数据的相关系数计算;
*   kendall: 针对无序序列的相关系数，非正态分布的数据;
*   spearman: 非线性, 非正态分布的数据相关系数;

min\_periods: 样本最少的数据量;

##### pearson-皮尔森相关系数

整体相关系数

$$
ρ_x,_y = \frac{Cov(X, Y)}{σ_X σ_Y} = \frac{E[(X - μ_X)(Y - μ_Y)}{σ_X σ_Y}
$$


样本相关系数:
$$
γ = \frac{\frac{\sum_{i = 1}^{n}(X_i - \overline{X})(Y_i - \overline{Y})}{n - 1}}
{\frac{\sqrt{\sum_{i = 1}^{n}(X_i - \overline{X})^2} \sqrt{\sum_{i = 1}^{n}(Y_i - \overline{Y})^2}}{n - 1}} 
= \frac{\sum_{i = 1}^{n}(X_i - \overline{X})(Y_i - \overline{Y})}{\sqrt{\sum_{i = 1}^{n}(X_i - \overline{X})^2} \sqrt{\sum_{i = 1}^{n}(Y_i - \overline{Y})^2}}
\\ = \frac{1}{n - 1} * \sum_{i = 1}^{n} (\frac{X_i - \overline{X}}{σ_X})(\frac{Y_i - \overline{Y}}{σ_Y})
$$




X(ba) 为**样本** 的均值, σx 为**样本**的样本标准差.

示例:

```python
# 下半部分: sqrt(sum(x - x^))
def get_pow_sum_sqrt(data):
    temp = np.array([np.mean(data)] * len(data))
    temp = data - temp
    temp = temp * temp
    return math.sqrt(np.sum(temp))
# 上半部分
def get_above(data1, data2):
    d1 = data1 - np.array([np.mean(data1)] * len(data1))
    d2 = data2 - np.array([np.mean(data2)] * len(data2))
    return np.sum(d1 * d2)
    
data1 = np.array([0.2, 0, 0.6, 0.2]) 
data2 = np.array([0.3, 0.6, 0, 0.1])
res0 = get_above(data1, data2)
res1 = get_pow_sum_sqrt(data1)
res2 = get_pow_sum_sqrt(data2)
res = res0 / (res1 * res2) # 计算结果: -0.8510644963469902
```

使用pandas:

```python
df = pd.DataFrame([(.2, .3), (.0, .6), (.6, .0), (.2, .1)],
                  columns=['dogs', 'cats'])
df.corr(method="pearson")
```

计算结果:

> dogs	cats
> dogs	1.000000	-0.851064
> cats	-0.851064	1.000000