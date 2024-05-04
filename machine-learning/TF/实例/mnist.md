#### mnist 训练完整过程

##### 直接使用 784 * 10 + softmax 

```python
import tensorflow as tf
from tensorflow.keras.datasets import mnist

# 加载 MNIST 数据集
(x_train, y_train), (x_test, y_test) = mnist.load_data()

# 数据预处理
x_train = x_train.reshape(-1, 28 * 28) / 255.0
x_test = x_test.reshape(-1, 28 * 28) / 255.0
y_train = tf.keras.utils.to_categorical(y_train, num_classes=10)
y_test = tf.keras.utils.to_categorical(y_test, num_classes=10)

# 构建计算图
x = tf.placeholder(tf.float32, shape=[None, 784])
y = tf.placeholder(tf.float32, shape=[None, 10])

# 定义模型
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
logits = tf.matmul(x, W) + b
pred = tf.nn.softmax(logits)

# 定义损失函数和优化器
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(labels=y, logits=logits))
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01).minimize(cost)

# 定义评估指标
correct_pred = tf.equal(tf.argmax(pred, 1), tf.argmax(y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_pred, tf.float32))  # 先将 bool 类型 -> float(true:1, false:0)，再求和

# 创建会话
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())

    # 训练循环
    for epoch in range(10):
        avg_cost = 0.0
        total_batch = int(x_train.shape[0] / 100)

        # 在每个批次进行训练
        for i in range(total_batch):
            batch_x, batch_y = x_train[i * 100: (i + 1) * 100], y_train[i * 100: (i + 1) * 100]
            _, c = sess.run([optimizer, cost], feed_dict={x: batch_x, y: batch_y})
            avg_cost += c
        avg_cost /= total_batch
        # 在每个周期结束时打印损失和准确率
        print("Epoch:", epoch + 1, "Cost:", avg_cost)
        acc = sess.run(accuracy, feed_dict={x: x_test, y: y_test})
        print("Accuracy:", acc)
```

迭代10次训练结果：

> Epoch: 1 Cost: 1.1449549620846908
> Accuracy: 0.8557
> Epoch: 2 Cost: 0.6415749228000641
> Accuracy: 0.8746
> Epoch: 3 Cost: 0.5357044726113478
> Accuracy: 0.8834
> Epoch: 4 Cost: 0.4846347537388404
> Accuracy: 0.8886
> Epoch: 5 Cost: 0.45329119426508746
> Accuracy: 0.8921
> Epoch: 6 Cost: 0.43158616208781797
> Accuracy: 0.8957
> Epoch: 7 Cost: 0.4154149424657226
> Accuracy: 0.899
> Epoch: 8 Cost: 0.4027598800510168
> Accuracy: 0.9008
> Epoch: 9 Cost: 0.39250094038744765
> Accuracy: 0.9029
> Epoch: 10 Cost: 0.3839608982702096
> Accuracy: 0.9038

##### 使用全连接 784 * 256 * 256 * 10 + softmax

```python
import tensorflow as tf
from tensorflow.keras.datasets import mnist

# 加载 MNIST 数据集
(x_train, y_train), (x_test, y_test) = mnist.load_data()

# 数据预处理
x_train = x_train.reshape(-1, 28 * 28) / 255.0
x_test = x_test.reshape(-1, 28 * 28) / 255.0
y_train = tf.keras.utils.to_categorical(y_train, num_classes=10)
y_test = tf.keras.utils.to_categorical(y_test, num_classes=10)

learning_rate = 0.1
num_steps = 20
batch_size = 100
display_step = 1

# Network Parameters
n_hidden_1 = 256  # 1st layer number of neurons
n_hidden_2 = 256  # 2nd layer number of neurons
num_input = 784  # MNIST data input (img shape: 28*28)
num_classes = 10  # MNIST total classes (0-9 digits)

# tf Graph input
X = tf.placeholder("float", [None, num_input])
Y = tf.placeholder("float", [None, num_classes])

# Store layers weight & bias
weights = {
    'h1': tf.Variable(tf.random_normal([num_input, n_hidden_1])),
    'h2': tf.Variable(tf.random_normal([n_hidden_1, n_hidden_2])),
    'out': tf.Variable(tf.random_normal([n_hidden_2, num_classes]))
}
biases = {
    'b1': tf.Variable(tf.random_normal([n_hidden_1])),
    'b2': tf.Variable(tf.random_normal([n_hidden_2])),
    'out': tf.Variable(tf.random_normal([num_classes]))
}


# Create model
def neural_net(x):
    # Hidden fully connected layer with 256 neurons
    layer_1 = tf.add(tf.matmul(x, weights['h1']), biases['b1'])
    # Hidden fully connected layer with 256 neurons
    layer_2 = tf.add(tf.matmul(layer_1, weights['h2']), biases['b2'])
    # Output fully connected layer with a neuron for each class
    out_layer = tf.matmul(layer_2, weights['out']) + biases['out']
    return out_layer


# Construct model
logits = neural_net(X)

# Define loss and optimizer
loss_op = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=Y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate)
train_op = optimizer.minimize(loss_op)

# Evaluate model (with test logits, for dropout to be disabled)
correct_pred = tf.equal(tf.argmax(logits, 1), tf.argmax(Y, 1))
accuracy = tf.reduce_mean(tf.cast(correct_pred, tf.float32))

# Initialize the variables (i.e. assign their default value)
init = tf.global_variables_initializer()

# Start training
with tf.Session() as sess:
    # Run the initializer
    sess.run(init)
    total_batch = int(x_train.shape[0] / 100)
    for step in range(num_steps):
        avg_cost = 0.
        for i in range(total_batch):
            batch_x, batch_y = x_train[i * 100: (i + 1) * 100], y_train[i * 100: (i + 1) * 100]
            _, c = sess.run([train_op, loss_op], feed_dict={X: batch_x, Y: batch_y})
            avg_cost += c
        if (step % display_step) == 0:
            acc = sess.run(accuracy, feed_dict={X: batch_x, Y: batch_y})
            print("Step " + str(step) + ", Minibatch Loss= " + "{:.4f}".format(avg_cost) + ", Training Accuracy= " + "{:.3f}".format(acc))
    print("Optimization Finished!")

    # Calculate accuracy for MNIST test images
    print("Testing Accuracy:", sess.run(accuracy, feed_dict={X: x_test, Y: y_test}))
```

迭代30次结果：

> Step 0, Minibatch Loss= 128419.4511, Training Accuracy= 0.900
> Step 1, Minibatch Loss= 53355.4819, Training Accuracy= 0.930
> Step 2, Minibatch Loss= 38578.2596, Training Accuracy= 0.890
> Step 3, Minibatch Loss= 28950.1437, Training Accuracy= 0.880
> Step 4, Minibatch Loss= 21302.1276, Training Accuracy= 0.920
> Step 5, Minibatch Loss= 17117.6175, Training Accuracy= 0.890
> Step 6, Minibatch Loss= 13095.3418, Training Accuracy= 0.920
> Step 7, Minibatch Loss= 10201.1689, Training Accuracy= 0.900
> Step 8, Minibatch Loss= 8365.6931, Training Accuracy= 0.890
> Step 9, Minibatch Loss= 7343.8289, Training Accuracy= 0.900
> Step 10, Minibatch Loss= 6366.7122, Training Accuracy= 0.910
> Step 11, Minibatch Loss= 5715.9657, Training Accuracy= 0.930
> Step 12, Minibatch Loss= 5678.0348, Training Accuracy= 0.900
> Step 13, Minibatch Loss= 5045.3281, Training Accuracy= 0.920
> Step 14, Minibatch Loss= 4680.5843, Training Accuracy= 0.910
> Step 15, Minibatch Loss= 4778.0079, Training Accuracy= 0.890
> Step 16, Minibatch Loss= 4211.9010, Training Accuracy= 0.920
> Step 17, Minibatch Loss= 4087.9765, Training Accuracy= 0.940
> Step 18, Minibatch Loss= 3709.9377, Training Accuracy= 0.930
> Step 19, Minibatch Loss= 3363.2034, Training Accuracy= 0.930
> Step 20, Minibatch Loss= 3124.2873, Training Accuracy= 0.930
> Step 21, Minibatch Loss= 2831.2341, Training Accuracy= 0.920
> Step 22, Minibatch Loss= 2562.2567, Training Accuracy= 0.930
> Step 23, Minibatch Loss= 2426.4480, Training Accuracy= 0.920
> Step 24, Minibatch Loss= 2285.6822, Training Accuracy= 0.910
> Step 25, Minibatch Loss= 1851.0465, Training Accuracy= 0.910
> Step 26, Minibatch Loss= 1754.0511, Training Accuracy= 0.920
> Step 27, Minibatch Loss= 1594.8655, Training Accuracy= 0.940
> Step 28, Minibatch Loss= 1487.8761, Training Accuracy= 0.930
> Step 29, Minibatch Loss= 1438.6149, Training Accuracy= 0.920
> Optimization Finished!
> Testing Accuracy: 0.8498