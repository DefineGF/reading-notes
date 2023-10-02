#### 基础定义

```python
def softmax(X):
    X_exp = X.exp()
    partition = X_exp.sum(dim=1, keepdim=True)
    return X_exp / partition  # 这里应用了广播机制


def net(x):
    """
    定义模型
    """
    return softmax(torch.mm(x.view((-1, num_inputs)), W) + b)


def cross_entropy(y_hat, y):
    """
    交叉熵损失函数
    :return:
    """
    return - torch.log(y_hat.gather(1, y.view(-1, 1)))

def accuracy(y_hat, y) -> float:
    """
    计算准确率:
    (y_hat.argmax(dim=1) == y) 返回值类型是 ByteTensor，需要转换为 float类型的0 | 1值
    :return:
    """
    return (y_hat.argmax(dim=1) == y).float().mean().item()


def evaluate_accuracy(data_iter, net) -> float:
    """
    评价模型在整个数据集 data_iter 中的准确率
    """
    acc_sum, n = 0.0, 0
    for X, y in data_iter:
        acc_sum += (net(X).argmax(dim=1) == y).float().sum().item()
        n += y.shape[0]
    return acc_sum / n
```

> 注：
>
> torch.gather(dim, input):
>
> dim = 0: 表示列方向；dim = 1 表示行方向
>
> ```python
> data = torch.arange(3, 12).view(3, 3)
> '''
> tensor([[ 3,  4,  5],
>      [ 6,  7,  8],
>      [ 9, 10, 11]])
> '''
> index = torch.tensor([[2, 1, 0]])
> data.gather(0, index)
> '''
> tensor([[9, 7, 5]]) # 表示在列方向上：第0列，取index=2的元素；第1列，取index=1的元素；第2列，取index=0的元素；、
> '''
> data.gather(1, index)
> '''
> tensor([[5, 4, 3]]) # 表示在行方向上：由于 index 是行向量，因此取的都是第0行的，index=2, 1, 0 的元素
> '''
> 
> # 将index转换成3 * 1 的形式
> (data.gather(1, index.view(-1, 1))
> '''
> 其中： index.view 
> tensor([[5],
>      [7],
>      [9]])
> 最终结果：
> tensor([[5],
>      [7],
>      [9]]) # 表示在行方向上，第0行取index=2，第1行取index=1，第2行取index=0的元素
> '''
> ```



#### 训练模型

