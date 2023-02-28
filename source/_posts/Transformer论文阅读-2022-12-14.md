---
title: Transformer论文阅读
date: 2022-12-14 15:12:55
categories:
  - - 研究生
    - 论文
  - - Transformer
tags:
  - CV
  - 图像处理
  - 论文笔记
  - Transformer
  - 端到端
headimg:
  'https://z3.ax1x.com/2021/08/05/fego40.png'
mathjax: true
---

Attention Is All You Need
<!-- more -->

## 摘要

在主流的Seq2Seq模型中主要依赖于复杂的CNN或RNN,使用Encoder-Decoder架构连接起来,在这两个模块直接通常会使用一个注意力机制将其连接起来.

本文提出了更为简单的模型Transformer,该模型仅使用了注意力机制,并在机器翻译任务上取得了较好成绩.

该网络并行度更好,需要使用的时间训练更少

## Intro

该部分描述了RNN现存的缺陷,即RNN中隐藏状态$h_t$需要参考当前位置输入词$t$和上一个隐藏状态$h_{t-1}$,因此在训练的过程中无法并行.

描述了注意力机制在Encoder-Decoder中编解码器中早已使用的事实.

## 相关工作

之前有人提出使用CNN来代替RNN减少时序计算,但CNN对于长序列难以建模,因为CNN每次只看一个小窗口(例如3*3卷积核),如果需要增大视野,则需要使用更深的卷积层,但使用注意力机制则每一次都能看到所以的序列.

另外CNN可以设置很多输出通道,每个通道代表了某种模态,因此该论文提出了Multi-Head Attention,可以模拟CNN多输出通道的效果.

## 模型

目前效果很好的编码器-解码器框架会通过编码器将输入序列$(x_1, ...,x_n)$表示为一个同样长度的新序列$z=(z_1,...,z_n)$,然后解码器会用这一序列得到一个长为m的序列$(y_1,...,y_m)$,即过去时刻的输出会作为当前时刻的输入,称之为自回归

### 编码器

编码器使用6个Transformer编码块堆叠而成,每个块包含两个子层:

![image-20221214182743806](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214182743806.png)

为了残差连接的方便,每一个层的输出维度定位512

其中Norm使用的是Layer Normal,而不是CNN中常用的Batch Normal,两者最大的区别就是BN对每个feature做均值方差标准化,而LN对每个Batch做均值方差标准化.

因此BN层容易受到Batch_Size的影响

### 解码器

使用6个Transformer解码块堆叠而成,每个块包含三个子层:

![image-20221214190123379](E:\EnderBlogSource\EnderBlog\source\images\MachineLearning\image-20221214190123379.png)

由于在注意力机制中,注意力每次都需要看到完整的输入,而对于自回归解码器而言,解码器只能看到当前输入和上一些时刻的输出,因此此处使用Masked Multi-Head Attention来处理.

### 注意力

常见的注意力机制有两种:

1. 加形注意力机制,可以处理query和key不等长的情况
2. 点积注意力机制,与本文方法相同,除了除以$\sqrt{d_k}$以外

注意力函数是一个将query和一些key-value对做映射成一个输出的函数,集体来说,输出output是value的加权和,因此output具有和value一样的维度.

每一个value的权重是其对应的key和query的相似度计算而来

而Transformer中采用的注意力机制为Scaled Dot-Product Attention,其中query和key是等长的假设为$d_k$,value长度为$d_v$

计算时对key和query做内积,该直作为相似度量,也就是正交向量的相似度最小.计算完成后除以$\sqrt {d_k}$即向量长度,然后用一个softmax得到权重,然后我们将权重作用到value上就能得到输出了.

$Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V$

其中QKV均可不止一个,即均可为矩阵

本文在点积注意力机制的基础上除以$\sqrt {d_k}$的目的是避免模型对长序列的倾向.

而带有掩码的Mask Multi-Head Attention就是当:

t时刻的$query_t$计算时,只能考虑$k_1,...,k_{t-1}$因为当前时刻还没有之后的数据,但是注意力机制是所有数据同时输入的.

### 多头注意力

因此在带有掩码的多头注意力机制中,我们先计算出Q和K的矩阵乘法,并除以$\sqrt{d_k}$然后对于$query_t$和$k_{t-1}$之后的值换成一个非常大的负数,这样这么大的负数在softmax之后得到的用于与V相乘的权重将会趋近于0

本文还提出,与其只做一个单个的注意力函数,不如把QKV投影到低维,并投影h次然后再做h次的注意力函数,然后每一个函数的输出合并到一起,然后投影回来得到最终输出,即多头注意力机制表现如下:

$MultiHead(Q,K,V) = Concat(head_1, head_2, ...,head_h)W^O$

$where \ head_i = Attention(QW^Q_i, KW^K_i, VW^V_i)$

实际使用中用到的为8头注意力机制,投影维度为单注意力机制的输出维度/h

### FFN

该层即为全连接层的作用，只不过是对每一个词做MLP：

$FFN(x) = max(0, xW_1+b_1)W_2 + b_2$

其中$W_1$会将512维度投影为1024维，$W_2$则是将1024维投影回512

### Embeddings

该层的目的在于对于一个词，学习一个长度为d的向量来表示这个词，本文d为512，transformer中编码器需要一个，解码器需要一个，最后softmax前的线性层也需要一个，文中三处的权重都是一样的，并且权重乘以$\sqrt{d}$,因为在Embedding学习的过程中会把L2范式学到过于小，那么维度越大，学习到的权重就会变小，但之后加上Positional Encoding时，其不会随着长度变长，因此需要保证其与Positional Encoding在一个尺度上。

### Positional Encoding

由于注意力机制不会考虑语序的信息，因此需要手动的将语序信息加入输入。Transformer中使用cos、sin函数处理：

$PE_{(pos,2i)}=sin(pos/10000^{2i/d_model})$

$PE_{(pos,2i+1)}=cos(pos/10000^{2i/d_model})$

## 结论

该论文将所有循环层都替换为了**多头自注意力层**,使得模型具有了更好的并行性,使得该方法训练时间更短
