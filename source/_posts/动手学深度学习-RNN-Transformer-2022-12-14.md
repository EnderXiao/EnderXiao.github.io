---
title: 动手学深度学习-RNN-Transformer
date: 2022-12-14 11:12:39
tags:
  - 环境搭建
  - PyTorch
  - Python
  - RNN
  - 编码解码架构
categories:
  - - 机器学习
  - - 硕士研究生
    - 机器学习入门
    - RNN
    - Transformer
    - 注意力机制
    - 自注意力机制
    - 端到端
    - 编码解码架构
mathjax: true
---
深度学习常用循环神经网络（RNN）与Transformer（纯注意力编码解码网络）
<!-- more -->

## Transformer

### 架构

- Transformer是基于编码器-解码器架构来处理序列对的
- 与带有注意力的seq2seq不同,Transformer是纯基于注意力的

![image-20221214111740887](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214111740887.png)

![image-20221214111854081](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214111854081.png)

### 多头注意力(Multi-head Attention)

实践过程中我们希望当给定相同的*查询*,*键*,*值*的时候,可以**基于相同的注意力机制学习到不同的行为,然后将不同的行为作为知识组合起来**,捕获序列内各种范围的依赖关系.

对于同一key,value,query,希望抽取不同的信息:

- 例如短距离关系和长距离关系

我们可以用独立学习得到的$h$组不同的*线性投影(linear projections)*来变换查询,键,值

多头注意力使用h个独立的注意力池化

- 合并各个头(head)输出得到最终输出

![image-20221214112554249](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214112554249.png)

数学上我们可以用如下形式来表示多头注意力机制:

给定查询$q \in R^{d_q}$,键$k \in R^{d_k}$,值$v \in R^{d_v}$,每个注意力头$h_i(i=1,...,h)$的计算方法为:

$h_i = f(W_i^{(q)}q, W_i^{(k)}k, W_i^{(v)}v) \in R^{p_v}$

其中$W_i^{(q)} \in R^{p_q\times d_q},W_i^{(k)} \in R^{p_k \times d_k}, W_i^{(v)} \in R^{p_v \times d_v}$为需要学习的参数,以及代表注意力汇聚的函数f.

f可以是**加性注意力**和**缩放点积注意力**

然后多头注意力的输出需要经过另一个线性转换,它对应着h个头连接后的结果,因此其可学习参数是$W_o \in R^{p_o \times hp_{v}}$:

$W_o \left[ \begin{matrix} h_1 \\ . \\ . \\ . \\ h_h \end{matrix} \right] \in R^{p_o}$

### 带有掩码的多头注意力

**解码器对序列中一个元素输出时,不应该考虑该元素之后的元素**,因此需要引入掩码来实现这一特性:

特就是计算$x_i$输出时,假装当前序列长度为i

### 基于位置的前馈网络(positionwise FFN)

> 事实上可以理解为一个全连接层

作用

- 将输入形状由$(b,n,d)$变换为$(bn,d)$
- 作用两个全连接层
- 输出形状由$(bn,d)$变化回$(b,n,d)$
- 等价于两层核窗口为1的一位卷积层

其中b表示batch size, n表示序列长度, d表示维度

### 层归一化(Add & norm)

批量归一化是对每个特征/通道离元素进行归一化,但其不适合序列长度会变化的NLP任务

因此提出层归一化

层归一化对每个样本里的元素进行归一化

二者的区别如图所示:

![image-20221214121750990](C:\Users\12865\AppData\Roaming\Typora\typora-user-images\image-20221214121750990.png)

对于一个Batch_size=b,feature_size=d,序列长度=len的一组数据,BN层的作法是将其在**特征维度**上进行归一化,而LN层则是在**Batch维度**上进行归一化,这样使得长度更稳定.

该层结构如图所示:

![image-20221214121530459](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214121530459.png)

其中ADD即表示**残差连接**

### 信息传递

假设编码器中的输出$y_1,...,y_n$,将其作为解码器中第i个Transformer块中Multi-head Attention中的key和value,它的query来自目标序列(即此处为**非自注意力机制**)

这意味着编码器和解码器中块的个数和输出维度都一样.

![image-20221214122551960](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214122551960.png)

### 预测

预测第$t+1$个输出时,解码器中输入前t个预测值

在自注意力中,前t个预测值作为key和value,第t个预测值还作为query

![image-20221214122923756](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214122923756.png)

### 总结

- Transformer是一个纯使用注意力的编码-解码器
- 编码器和解码器都有n个transformer块
- 每个块中使用多头(自)注意力,基于位置的前馈网络,和层归一化

### 代码实验

先来实现Multi-Head attention

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, key_size, query_size, value_size, num_hiddens, num_heads, dropout, bias=False, **kwargs):
        super(MultiHeadAttention, self).__init__(**kwargs)
        # TODO: 头数
        self.num_heads = num_heads
        # TODO: 注意力层
        self.attention = DotProductAttention(dropout)
        # TODO: 学习query, key value的投影矩阵(即线性层),以及
        self.W_q = nn.Linear(query_size, num_hiddens, bias=bias)
        self.W_k = nn.Linear(key_size, num_hiddens, bias=bias)
        self.W_v = nn.Linear(value_size, num_hiddens, bias=bias)
        self.W_o = nn.Linear(num_hiddens, num_hiddens, bias=bias)

    def forward(self, queries, keys, values, valid_lens):
        # queries，keys，values的形状:
        # (batch_size，查询或者“键－值”对的个数，num_hiddens)
        # valid_lens　的形状:
        # (batch_size，)或(batch_size，查询的个数)
        # 经过变换后，输出的queries，keys，values　的形状:
        # (batch_size*num_heads，查询或者“键－值”对的个数，
        # num_hiddens/num_heads)
        queries = transpose_qkv(self.W_q(queries), self.num_heads)
        keys = transpose_qkv(self.W_k(keys), self.num_heads)
        values = transpose_qkv(self.W_v(values), self.num_heads)

        if valid_lens is not None:
            # 在轴0，将第一项（标量或者矢量）复制num_heads次，
            # 然后如此复制第二项，然后诸如此类。
            valid_lens = torch.repeat_interleave(
                valid_lens, repeats=self.num_heads, dim=0)

        # output的形状:(batch_size*num_heads，查询的个数，
        # num_hiddens/num_heads)
        output = self.attention(queries, keys, values, valid_lens)

        # output_concat的形状:(batch_size，查询的个数，num_hiddens)
        output_concat = transpose_output(output, self.num_heads)
        return self.W_o(output_concat)
```

## 注意力机制

注意力机制是源于19世纪90年代威廉詹姆斯提出的*双组件（two-component）*框架，该研究展示了注意力是如何作用于视觉世界中的，

该框架中受试者于*非自主性提示*和*自主性提示*有选择性地引导注意力的焦点。

- 非自主性提示是基于环境中物体的突出性和易见性
  - 例如我们有五件物品：报纸、论文、咖啡、笔记本、一本书
  - 那么如果在这些物品中，数、笔记本、论文、报纸这些纸质品均为黑白印刷，而咖啡杯为醒目的红色，那么咖啡杯在这个环境中则是突出的、显眼的
  - 所以我们会将视力最敏锐的地方放在咖啡上
- 自主性提示基于受试者的主观意愿推动，选择的力量也就更强大
  - 喝完咖啡后我们变得兴奋，更想读书了，于是将注意力聚焦于书本。
  - 这与之前提到的，由于突出性导致的注意咖啡不同，此次选择书本是收到了认识和意识的控制，因此注意力在基于自主性提示去辅助选择时将更为谨慎。

### 查询、键和值

那么如何利用上述两种注意力提示，用神经网络来设计注意力机制的框架。

#### 非自主性提示

首先考虑只包含非自主性提示，那么想将选择偏向于感官**输入**，则可以简单地使用参数化的全连接层，甚至非参数化的MaxPooling和AvgPooling

#### 自主性提示

在注意力机制的背景下，自主性提示被成为*查询（query）*。给定任何查询，注意力机制通过*注意力汇聚（attention pooling）*将选择引导至*感官输入（sensory inputs， 例如中间特征表示）*

而在注意力机制中，这些感官输入被成为*值(value)*，而每一个值都有一个与之对应的*键(key)*

键可以想象为感官输入的非自自主提示，可以通过设计注意力汇聚的方式， 便于给定的查询（自主性提示）与键（非自主性提示）进行匹配， 这将引导得出最匹配的值（感官输入）。

![image-20221223145603038](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223145603038.png)

#### 注意力可视化

平均汇聚层可以被视为输入的加权平均值， 其中各输入的权重是一样的。 实际上，注意力汇聚得到的是加权平均的总和值， 其中权重是在给定的查询和不同的键之间计算得出的。

```python
import torch
from d2l import torch as d2l


def show_heatmaps(matrices, xlabel, ylabel, titles=None, figsize=(2.5, 2.5), cmap='Reds'):
    """显示矩阵热图"""
    d2l.use_svg_display()
    num_rows, num_cols = matrices.shape[0], matrices.shape[1]
    fig, axes = d2l.plt.subplots(
        num_rows, num_cols, figsize=figsize, sharex=True, sharey=True, squeeze=False)

    for i, (row_axes, row_matrices) in enumerate(zip(axes, matrices)):
        for j, (ax, matrix) in enumerate(zip(row_axes, row_matrices)):
            pcm = ax.imshow(matrix.detach().numpy(), cmap=cmap)
            if i == num_rows - 1:
                ax.set_xlabel(xlabel)
            if j == 0:
                ax.set_ylabel(ylabel)
            if titles:
                ax.set_title(titles[j])

        fig.colorbar(pcm, ax=axes, shrink=0.6)


attention_weight = torch.eye(10).reshape((1, 1, 10, 10))
show_heatmaps(attention_weight, xlabel='Keys', ylabel='Queries')
d2l.plt.show()

```

如图所示的我们使用了一个简单的例子来显示query和key，本例中只有当query和key相同时，注意力权重为1，否则为0。

![image-20221223152507097](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223152507097.png)

### 注意力汇聚实验：Nadaraya-Watson核回归

首先我们生成一个人工数据集，使用如下函数来生成：为：

$y_i = 2sin(x_i) + x_i^{0.8} + \epsilon$

```python
n_train = 50  # 训练样本数
x_train, _ = torch.sort(torch.rand(n_train) * 5)   # 排序后的训练样本

def f(x):
    return 2 * torch.sin(x) + x**0.8

y_train = f(x_train) + torch.normal(0.0, 0.5, (n_train,))  # 训练样本的输出
x_test = torch.arange(0, 5, 0.1)  # 测试样本
y_truth = f(x_test)  # 测试样本的真实输出
n_test = len(x_test)  # 测试样本数
```

Nadaraya和Watson在很早之前就提出了一个想法，根据输入的位置对输出$y_i$进行加权：

$f(x) = \sum_{i=1}^n \frac{K(x-x_i)}{\sum_{j=1}^n K(x-x_j)}y_i$

其中K为核函数，如果我们从注意力机制的角度来重写上式，那我们可以得到一个注意力汇聚的通用形式：

$f(x) = \sum_{i=1}^n \alpha (x, x_i)y_i$

其中$x$时查询，$(x_i,y_i)$时键值对，注意力汇聚则是$y_i$的加权平均。$x$与$x_i$之间的关系建模为注意力权重。

#### 非参数注意力汇聚

下面我们使用高斯核函数作为K来进行实验：

$K(u) = \frac{1}{\sqrt{2\pi}}exp(-\frac{u^2}{2})$

将其带入可得：

$f(x) = \sum_{i=1}^{n}\frac{exp(-\frac{1}{2}(x-x_i)^2)}{\sum_{j=1}^n exp(-\frac{1}{2} (x-x_i)^2)}y_i \\ = \sum_{i=1}^n softmax(-\frac{1}{2}(x-x_i)^2)y_i $

```python
# X_repeat的形状:(n_test,n_train),
# 每一行都包含着相同的测试输入（例如：同样的查询）
X_repeat = x_test.repeat_interleave(n_train).reshape((-1, n_train))
# x_train包含着键。attention_weights的形状：(n_test,n_train),
# 每一行都包含着要在给定的每个查询的值（y_train）之间分配的注意力权重
attention_weights = nn.functional.softmax(-(X_repeat - x_train)**2 / 2, dim=1)
# y_hat的每个元素都是值的加权平均值，其中的权重是注意力权重
y_hat = torch.matmul(attention_weights, y_train)
plot_kernel_reg(y_hat)
d2l.plt.show()
```

得到拟合结果如下：

![image-20221223171917321](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223171917321.png)

现在来观察注意力的权重。 这里测试数据的输入相当于查询，而训练数据的输入相当于键。 因为两个输入都是经过排序的，因此由观察可知“查询-键”对越接近， 注意力汇聚的注意力权重就越高。![image-20221223172401720](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223172401720.png)

#### 带参数注意力汇聚

我们可以引入一些可学习的参数来使得我们的模型具有更好的泛化能力，例如对于高斯核函的注意力汇聚函数中，让距离乘以参数：

$f(x) = \sum_{i=1}^n softmax(-\frac{1}{2}((x-x_i)w)^2)y_i $

那么我们可以构建一个如下所示的模型：

```python
class NWKernelRegression(nn.Module):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.w = nn.Parameter(torch.rand((1,), requires_grad=True))

    def forward(self, queries, keys, values):
        # queries和attention_weights的形状为(查询个数，“键－值”对个数)
        queries = queries.repeat_interleave(
            keys.shape[1]).reshape((-1, keys.shape[1]))
        self.attention_weights = nn.functional.softmax(
            -((queries - keys) * self.w)**2 / 2, dim=1)
        # values的形状为(查询个数，“键－值”对个数)
        return torch.bmm(self.attention_weights.unsqueeze(1),
                         values.unsqueeze(-1)).reshape(-1)
```

接下来我们可以对模型使用一些策略来训练，例如使用平方损失函数和随机梯度下降：

```python
# X_tile的形状:(n_train，n_train)，每一行都包含着相同的训练输入
X_tile = x_train.repeat((n_train, 1))
# Y_tile的形状:(n_train，n_train)，每一行都包含着相同的训练输出
Y_tile = y_train.repeat((n_train, 1))
# keys的形状:('n_train'，'n_train'-1)
keys = X_tile[(1 - torch.eye(n_train)).type(torch.bool)].reshape((n_train, -1))
# values的形状:('n_train'，'n_train'-1)
values = Y_tile[(1 - torch.eye(n_train)).type(torch.bool)
                ].reshape((n_train, -1))

net = NWKernelRegression()
loss = nn.MSELoss(reduction='none')
trainer = torch.optim.SGD(net.parameters(), lr=0.5)
animator = d2l.Animator(xlabel='epoch', ylabel='loss', xlim=[1, 5])

for epoch in range(5):
    trainer.zero_grad()
    l = loss(net(x_train, keys, values), y_train)
    l.sum().backward()
    trainer.step()
    print(f'epoch {epoch + 1}, loss {float(l.sum()):.6f}')
    animator.add(epoch + 1, float(l.sum()))
```

最终得到的拟合效果如下图所示：

![image-20221223175154772](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223175154772.png)

接着来看一眼他的注意力权重：

与非参数的注意力汇聚模型相比， 带参数的模型加入可学习的参数后， 曲线在注意力权重较大的区域变得更不平滑。

![image-20221223175238910](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221223175238910.png)

### 自注意力与位置编码

#### 自注意力

- 给定序列$x_1, ..., x_n, \forall x_i \in R^d $
- 子注意力池化层将$x_i$当作key，value，query来对序列抽取特征得到$y_1,...,y_n$其中$y_i = f(x_i,(x_1,x_1),...,(x_n,x_n)) \in R^d$

#### 自注意力与CNN、RNN对比

|            | CNN      | RNN     | 自注意力 |
| ---------- | -------- | ------- | -------- |
| 计算复杂度 | O(knd^2) | O(nd^2) | O(n^2d)  |
| 并行度     | O(n)     | O(1)    | O(n)     |
| 最长路径   | O(n/k)   | O(n)    | O(1)     |

接下来比较下面几个架构，目标都是将由n个词元组成的序列映射到另一个长度相等的序列，其中的每个输入词元或输出词元都由d维向量表示。具体来说，将比较的是卷积神经网络、循环神经网络和自注意力这几个架构的计算复杂性、顺序操作和最大路径长度。由于顺序操作会妨碍并行计算，而任意的序列位置组合之间的路径越短，则能更轻松地学习序列中的远距离依赖关系

![image-20221228165805138](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221228165805138.png)

如图所示卷积神经网络由于收到卷积核大小的限制，每个神经元都将看到小范围相邻的几个元素，而随着层数的增加，卷积神经网络的感受野就会更大。

使用卷积神经网络处理序列时，我们设置序列长度为n，输入和输出通道数均为d，因此卷积层的计算复杂度为$O(knd^2)$ 卷积神经网络是分层的，因此为有O(1)个顺序操作， 最大路径长度为O(n/k)。 例如，x1和x5处于图中卷积核大小为3的双层卷积神经网络的感受野内。

当更新循环神经网络的隐状态时， d×d权重矩阵和d维隐状态的乘法计算复杂度为$O(d^2)$。 由于序列长度为n，因此循环神经网络层的计算复杂度为$O(nd^2)$。 根据 上图所示， 有O(n)个顺序操作无法并行化，最大路径长度也是O(n)。

在自注意力中，查询、键和值都是n×d矩阵。 考虑 使用缩放的点积注意力， 其中n×d矩阵乘以d×n矩阵。 之后输出的n×n矩阵乘以n×d矩阵。 因此，自注意力具有$O(n^2d)$计算复杂性。 正如图中所示， 每个词元都通过自注意力直接连接到任何其他词元。 因此，有O(1)个顺序操作可以并行计算， 最大路径长度也是O(1)。

因此，卷积神经网络和自注意力都拥有并行计算的优势， 而且自注意力的最大路径长度最短。 但是因为其计算复杂度是关于序列长度的二次方，所以在很长的序列中计算会非常慢。

##### 代码实现

```python
import math
import torch
from torch import nn
from d2l import torch as d2l

num_hiddens, num_heads = 100, 5
attention = d2l.MultiHeadAttention(num_hiddens, num_hiddens, num_hiddens,
                                   num_hiddens, num_heads, 0.5)
print(attention.eval())

batch_size, num_queries, valid_lens = 2, 4, torch.tensor([3, 2])
X = torch.ones((batch_size, num_queries, num_hiddens))
print(attention(X, X, X, valid_lens).shape)
```



#### 位置编码

- 跟CNN/RNN不同，自注意力并没有记录位置信息

- 位置编码将位置信息注入到输入中
  - 假设长度为n的序列是$X \in R^{n \times d}$，那么使用位置编码矩阵$P \in R^{n \times d}$来输出$X + P$作为自编码输入
- P的元素如下计算：
  - $p_{i,2j} = sin(\frac{i}{10000^{2j/d}}), p_{i,2j+1} = cos(\frac{i}{10000^{2j/d}})$

![image-20221228172302090](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221228172302090.png)

```python
class PositionalEncoding(nn.Module):
    """位置编码"""
    def __init__(self, num_hiddens, dropout, max_len=1000):
        super(PositionalEncoding, self).__init__()
        self.dropout = nn.Dropout(dropout)
        # 创建一个足够长的P
        self.P = torch.zeros((1, max_len, num_hiddens))
        X = torch.arange(max_len, dtype=torch.float32).reshape(
            -1, 1) / torch.pow(10000, torch.arange(
            0, num_hiddens, 2, dtype=torch.float32) / num_hiddens)
        self.P[:, :, 0::2] = torch.sin(X)
        self.P[:, :, 1::2] = torch.cos(X)

    def forward(self, X):
        X = X + self.P[:, :X.shape[1], :].to(X.device)
        return self.dropout(X)
 
encoding_dim, num_steps = 32, 60
pos_encoding = PositionalEncoding(encoding_dim, 0)
pos_encoding.eval()
X = pos_encoding(torch.zeros((1, num_steps, encoding_dim)))
P = pos_encoding.P[:, :X.shape[1], :]
d2l.plot(torch.arange(num_steps), P[0, :, 6:10].T, xlabel='Row (position)',
         figsize=(6, 2.5), legend=["Col %d" % d for d in torch.arange(6, 10)])
d2l.plt.show()

P = P[0, :, :].unsqueeze(0).unsqueeze(0)
d2l.show_heatmaps(P, xlabel='Column (encoding dimension)',
                  ylabel='Row (position)', figsize=(3.5, 4), cmap='Blues')

d2l.plt.show()
```



##### 绝对位置信息

该位置编码的思路有点类似计算机中使用二进制编码表示数字的方法

```powershell
0 in binary is 000
1 in binary is 001
2 in binary is 010
3 in binary is 011
4 in binary is 100
5 in binary is 101
6 in binary is 110
7 in binary is 111
```

其中二进制中低位数由0变为1的频率更快。而在之前使用的位置编码中，使用三角函数在编码维度上降低频率。由于输出是浮点数，因此此类线序表示会比二进制表示更节省空间。

![image-20230104150005130](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20230104150005130.png)

 

## RNN

### 序列模型

- 具有时序结构
  - 例如电影评分随时间变化而变化
- 音乐、语言、文本、视频都是连续的
- 人的交互是连续的

#### 统计工具

- 假设在时间$t$观察到$x_t$，那么得到T个不独立的随机变量：$(x_1,...,x_t) \sim p(X)$
- 使用条件概率展开得到：
  - $p(a,b) = p(a)p(b|a) = p(b)p(a|b)$
- 则:$p(X) = p(x_1)\cdot p(x_2|x_1)\cdot p(x_3|x_1,x_2)\cdot ... \cdot p(x_T|x_1,...,x_{T-1})$
- 或者反序计算：$p(X) = p(x_T)\cdot p(x_{T-1}|x_T)\cdot p(x_{T-2}|x_{T-1},x_T)\cdot ... \cdot p(x_1|x_2,...,x_T)$
- 对于条件概率建模：$p(x_t|x_1,...,x_{t-1}) = p(x_t|f(x_1,...,x_{t-1}))$
  - 我们已知前t-1个时刻的信息，那么要根据这些信息建模来预测t时刻的信息，这种方式也成为**自回归模型**

### 方案A-马尔科夫假设

假设当前数据只与$\tau$个过去数据点相关

$p(x_t|x_1,...,x_{t-1}) = p(x_t|x_{t-\tau},...,x_{t-1})\\ =p(x_t|f(x_{t-\tau},...,x_{t-1}))$

### 方案B-潜变量模型

引入潜变量$h_t$来表示过去信息$h_t = f(x_1,...,x_{t-1})$

这样$x_t = p(x_t|h_t)$

### 文本预处理

- 将文本中的标点转换为空格
- 按行分开
- 将文本拆分为单词或token
- 构建一个字典，从字符串映射到一个从0开始的数字索引
- 然后将每一行中的每一个token转换为索引输出，称为corpus

### 语言模型

给定一个文本序列$x_1,...,x_T$，语言模型的目标是估计联合概率$p(x_1,...,x_T)$

应用包括：

- 做预训练模型（BERT，GPT-3）
- 生成文本
- 本段多个序列中哪个更常见

例如使用序列长度为2的模型，预测：

$p(x,x') = p(x)p(x'|x) = \frac{n(x)}{x} \frac{n(x,x')}{n(x)}=\frac{n(x,x')}{n}$

上述公式中n是总词数(文本数据中token的总数)，n(x)表示x在文本数据中出现的次数；$n(x,x')$表示连续单词$(x,x')$在文本数据中出现的次数

但当序列很长的时候，因为文本量不够大，很可能$n(x_1,...,x_T) \leq 1$

于是可以使用马尔科夫假设缓解这个问题：

- N元语法：即认为马尔可夫假设中的$\tau = N-1$

![image-20230426210051765](C:\Users\12865\AppData\Roaming\Typora\typora-user-images\image-20230426210051765.png)

$\to {\hat{}} \to$

## GRU

由于RNN处理长序列时，隐藏变量由于隐藏了非常多的信息，那么其对于很久以前的信息表示的就不够明确。

为了解决这一问题，我们发现事实上一个序列中并不是每个观察值都同等重要，模型只需要记住相关的观察重点。

而RNN并没有这一机制，RNN将所有单元都视为同等重要。

因此提出了两个门控单元：

- 能关注的机制（更新门）
- 能遗忘的机制（重置门）

### 门

![image-20230427162647289](C:\Users\12865\AppData\Roaming\Typora\typora-user-images\image-20230427162647289.png)

在计算每个单元的隐藏状态时，计入两个门控单元，对H和X进行计算，其中$\sigma$表示sigmoid函数。

### 候选隐藏状态

![image-20230427163238632](C:\Users\12865\AppData\Roaming\Typora\typora-user-images\image-20230427163238632.png)

候选隐状态实际上是在普通RNN的隐状态中使用R与上一时刻的隐状态相乘，由于R通过sigmoid函数计算得到，因此其中的数字位于$[0,1]$之间，即可以被理解为R学习到了上一时刻的隐状态中，哪些比较重要（值接近1）哪些不太重要可以遗忘（值接近0）

### 隐状态

![image-20230427163202490](C:\Users\12865\AppData\Roaming\Typora\typora-user-images\image-20230427163202490.png)

使用Z控制改单元是否忽略掉当前的元素Xt，即当Zt接近1的时候，输出隐状态直接使用上一次的隐状态，即不包含当前元素Xt的信息，而Zt接近0的时候则近似等于普通RNN
