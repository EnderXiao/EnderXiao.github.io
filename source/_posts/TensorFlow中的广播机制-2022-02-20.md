---
title: TensorFlow中的广播机制
date: 2022-02-20 22:00:05
categories:
  - - 研究生
  - - Python
    - TensorFlow
tags:
  - TensorFlow
  - 广播机制
headimg:
  'https://z3.ax1x.com/2021/08/16/fWOnf0.jpg'
plugins:
  - mathjax
---

## [转载：tensorflow的广播机制](https://www.cnblogs.com/hrnn/p/13552497.html)

TensorFlow支持广播机制（Broadcast）

TensorFlow支持广播机制（Broadcast），可以广播元素间操作(elementwise operations)。正常情况下，当你想要进行一些操作如加法，乘法时，你需要确保操作数的形状是相匹配的，如：你不能将一个具有形状[3, 2]的张量和一个具有[3,4]形状的张量相加。但是，这里有一个特殊情况，那就是当你的其中一个操作数是**一个具有单独维度(singular dimension)的张量**的时候，TF会隐式地在它的单独维度方向填满(tile)，以确保和另一个操作数的形状相匹配。所以，对一个[3,2]的张量和一个[3,1]的张量相加在TF中是合法的。（译者：这个机制继承自numpy的广播功能。其中**所谓的单独维度就是一个维度为1，或者那个维度缺失**）

```python
import tensorflow as tf


a = tf.constant([[1., 2.], [3., 4.]])
b = tf.constant([[1.], [2.]])
# c = a + tf.tile(b, [1, 2])
c = a + b
```

广播机制允许我们在隐式情况下进行填充（tile），而这可以使得我们的代码更加简洁，并且更有效率地利用内存，因为我们不需要另外储存填充操作的结果。一个可以表现这个优势的应用场景就是在结合具有不同长度的特征向量的时候。为了拼接具有不同长度的特征向量，我们一般都先填充输入向量，拼接这个结果然后进行之后的一系列非线性操作等。这是一大类神经网络架构的共同套路(common pattern)

```python
a = tf.random_uniform([5, 3, 5])
b = tf.random_uniform([5, 1, 6])


# concat a and b and apply nonlinearity
tiled_b = tf.tile(b, [1, 3, 1])
c = tf.concat([a, tiled_b], 2)
d = tf.layers.dense(c, 10, activation=tf.nn.relu)
```

 

但是这个可以通过广播机制更有效地完成。我们利用事实f(m(x+y))=f(mx+my)f(m(x+y))=f(mx+my)，简化我们的填充操作。因此，我们可以分离地进行这个线性操作，利用广播机制隐式地完成拼接操作。

```python
pa = tf.layers.dense(a, 10, activation=None)
pb = tf.layers.dense(b, 10, activation=None)
d = tf.nn.relu(pa + pb)
```

事实上，这个代码足够通用，并且可以在具有抽象形状(arbitrary shape)的张量间应用：

```python
def merge(a, b, units, activation=tf.nn.relu):
    pa = tf.layers.dense(a, units, activation=None)
    pb = tf.layers.dense(b, units, activation=None)
    c = pa + pb
    if activation is not None:
        c = activation(c)
    return c
```

一个更为通用函数形式如上所述：

目前为止，我们讨论了广播机制的优点，但是同样的广播机制也有其缺点，隐式假设几乎总是使得调试变得更加困难，考虑下面的例子：

```python
a = tf.constant([[1.], [2.]])
b = tf.constant([1., 2.])
c = tf.reduce_sum(a + b)
```

你猜这个结果是多少？如果你说是6，那么你就错了，答案应该是12.这是因为当两个张量的阶数不匹配的时候，在进行元素间操作之前，TF将会自动地在更低阶数的张量的第一个维度开始扩展，所以这个加法的结果将会变为[[2, 3], [3, 4]]，所以这个reduce的结果是12. 

（译者：答案详解如下，第一个张量的shape为[2, 1]，第二个张量的shape为[2,]。因为从较低阶数张量的第一个维度开始扩展，所以应该将第二个张量扩展为shape=[2,2]，也就是值为[[1,2], [1,2]]。第一个张量将会变成shape=[2,2]，其值为[[1, 1], [2, 2]]。） 

解决这种麻烦的方法就是尽可能地显示使用。我们在需要reduce某些张量的时候，显式地指定维度，然后寻找这个bug就会变得简单：

```python
a = tf.constant([[1.], [2.]])
b = tf.constant([1., 2.])
c = tf.reduce_sum(a + b, 0)
```

这样，c的值就是[5, 7]，我们就容易猜到其出错的原因。一个更通用的法则就是总是在reduce操作和在使用`tf.squeeze`中指定维度。

## 我遇到的情况

在我尝试使用广播机制时，对于如下测试用例：

```python
# 广播机制, a和b规模不同，但可以进行加法
print("---------------------广播机制---------------------")
# a = tf.reshape(tf.range(8), (4, 2, 1))
# b = tf.reshape(tf.range(4), (2, 1, 2))
# print(a)
# print(b)
# print(a + b)
x = tf.constant([[[1], [1]], [[2], [2]], [[3], [3]], [[4], [4]]])
y = tf.constant([[[1, 1]], [[2, 2]]])
print(x + y)
```

将会报如下错误：

```shell
raise core._status_to_exception(e) from None  # pylint: disable=protected-access
tensorflow.python.framework.errors_impl.InvalidArgumentError: Incompatible shapes: [4,2,1] vs. [2,1,2] [Op:AddV2]
```

可见只有维度值为1的维度才会被复制，就算上例中的x第一个维度时y的两倍。
