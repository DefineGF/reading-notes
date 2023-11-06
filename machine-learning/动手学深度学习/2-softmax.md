### 版本1

###### 批量梯度下降

```python
def sgd(params, lr, batch_size):
    """
    优化算法
    """
    with torch.no_grad():
        for param in params:
            param -= lr * param.grad / batch_size
            param.grad.zero_()
```



##### softmax

```python
def softmax(X):
    x_exp = torch.exp(X)
    part = x_exp.sum(1, keepdim=True)
    tmp = x_exp / part
    return tmp
```



##### 交叉熵

```python
def cross_entry(y_hat, y):
    """
    定义交叉熵损失函数:
    假设样本数为4，正确类别分别是 [1, 0, 1，2] = y
    y_hat[range(len(y_hat)), y] 表示从0 到 3 个样本中，分别获取下标为 1，0，1，2 的元素值
    """
    return -1 * torch.log(y_hat[range(len(y_hat)), y])
```



#### 精准度

```python
def accuracy(y_hat, y):
    """
    先将计算结果类型转换成标签类型；
    和标签比较之后，将比较结果（bool）转换为标签类型，再取和
    """
    if len(y_hat.shape) > 1 and y_hat.shape[1] > 1:
        y_hat = y_hat.argmax(axis=1)
    cmp = y_hat.type(y.dtype) == y
    return float(cmp.type(y.dtype).sum())


def eval_accuracy(net, data_iter):
    if isinstance(net, torch.nn.Module):
        net.eval()  # 评估模式：不计算梯度
    metric = Accumulator(2)
    with torch.no_grad():
        for x, y in data_iter:
            metric.add(accuracy(net(x), y), y.numel())  # y.numel()返回当前样本的总数量
    return metric[0] / metric[1]
```



#### 训练

```python
def train_epoch(net, train_iter, loss, updater):
    """
    返回：损失率，准确率
    """
    if isinstance(net, torch.nn.Module):
        net.train()
    metric = Accumulator(3)  # 损失总和，精准度总和，样本数
    for x, y in train_iter:
        y_hat = net(x)
        ls = loss(y_hat, y)
        if isinstance(updater, torch.optim.Optimizer):
            # torch 内置优化器和损失函数
            updater.zero_grad()
            ls.mean().backward()
            updater.step()
        else:
            ls.sum().backward()
            updater(x.shape[0])
        metric.add(float(ls.sum()), accuracy(y_hat, y), y.numel())
    return metric[0] / metric[2], metric[1] / metric[2]


def train_softmax(net, train_iter, test_iter, loss, num_epochs, updater):
    for epoch in range(num_epochs):
        train_metrics = train_epoch(net, train_iter, loss, updater)
        test_acc = eval_accuracy(net, test_iter)
        animator.add(epoch + 1, train_metrics + (test_acc,))
        train_loss, train_acc = train_metrics
        print("epoch: ", epoch, ", train_loss: ", train_loss, ", train_acc: ", train_acc)
```



#### 调用

```python
lr = 0.1  # 学习率

def updater(batch_size):
    return sgd([W, b], lr, batch_size)


if __name__ == "__main__":
    num_epoch = 3  # 训练次数
    animator = Animator(xlabel='epoch', xlim=[1, num_epoch], ylim=[0.3, 0.9],
                        legend=['train loss', 'train acc', 'test acc'])
    train_softmax(net, train_iter, test_iter, cross_entry, num_epoch, updater)
    animator.show()
```



### 版本2

```python
from torch import nn
from my_softmax import *


# 初始化模型参数
def init_weights(m):
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)


def test():
    # 定义模型 并 初始化参数
    net = nn.Sequential(nn.Flatten(), nn.Linear(784, 10))
    net.apply(init_weights)

    # 定义损失函数
    # 当设置reduction='none' 时，CrossEntropyLoss函数将为每个样本返回一个损失值，而不是返回整个批次的平均损失。
    # 具体来说，对于输入的每个样本，CrossEntropyLoss函数都会计算一个损失值，并将这些损失值作为一个张量返回
    loss = nn.CrossEntropyLoss(reduction='none')

    # 定义优化器
    trainer = torch.optim.SGD(net.parameters(), lr=0.1)

    # 记载数据
    batch_size = 18
    train_itr, test_itr = DataLoader.load_data_fashion_mnist(batch_size)

    # 训练数据
    num_epochs = 10
    train_softmax(net, train_itr, test_itr, loss, num_epochs, trainer)
```

