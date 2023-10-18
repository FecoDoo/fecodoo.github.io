---
title: "Bidirectional LSTM"
date: 2020-11-08T21:24:50+08:00
draft: false
Tags: ["Development", "golang"]
Categories: "ML"
---

## 为什么用双向 LSTM？

单向的 RNN，是根据前面的信息推出后面的，但有时候只看前面的词是不够的，
例如，

我今天不舒服，我打算\_\_\_\_一天。

只根据‘不舒服‘，可能推出我打算‘去医院‘，‘睡觉‘，‘请假‘等等，但如果加上后面的‘一天‘，能选择的范围就变小了，‘去医院‘这种就不能选了，而‘请假‘‘休息‘之类的被选择概率就会更大。

## 什么是双向 LSTM？

双向卷积神经网络的隐藏层要保存两个值， A 参与正向计算， A' 参与反向计算。
最终的输出值 y 取决于 A 和 A'：


## 结构

我们已经看到了 Encoder-Decoder LSTM 的介绍中讨论的 LSTMs 输入序列的顺序的好处。

> 我们对源句子中颠倒词的改进程度感到惊讶。
>
> — Sequence to Sequence Learning with Neural Networks, 2014.

Bidirectional LSTMs 专注于通过输入和输出时间步长在向前和向后两个方向上获得最大的输入序列的问题。在实践中，该架构涉及复制网络中的第一个递归层，使得现在有两个并排的层，然后提供输入序列，作为输入到第一层并且提供输入序列到第二层的反向副本。这种方法是在不久前发展起来的一种用于改善循环神经网络（RNNs）性能的一般方法。

> 为了克服常规 RNN 的局限性...我们提出了一个双向递归神经网络（BRNN），可以使用所有可用的输入信息在过去和未来的特定时间帧进行训练。...我们的方法是将一个规则的 RNN 状态神经元分裂成两个部分，一部分负责正时间方向（正向状态），另外一个部分负责负时间方向（后向状态）。
>
> — Bidirectional Recurrent Neural Networks, 1997.

该方法以及被应用于 LSTM 循环神经网络。向前和向后提供整个序列是基于假设整个序列是可用的假设的。在使用矢量化输入时，在这个实践中通常是一个要求。然而，它可能会引起哲学上的关注，其中理想的时间步长是按顺序和及时（just-in-time）提供的。在语音识别领域中，双向地提供输入序列是合理的，因为有证据表明，在人类中，整个话语的上下文被用来解释所说的话而不是一个线性解释。

> ...依赖于乍一看是违反因果关系的未来知识。我们如何才能理解我们所听到的关于海没有说过的话呢？然而，人类的厅总就是这样做的。在未来的语境中，声音、词语乃至于整个句子都是毫无意义的。我们必须记住的是，任务之间的区别是真的在线的——在每个输入之后需要一个输出，以及在某些输入段的末尾只需要输出。
>
> — Framewise Phoneme Classiﬁcation with Bidirectional LSTM and Other Neural Network Architectures, 2005.

虽然 Bidirectional LSTMs 被开发用于语音识别，但是使用双向输入序列是序列预测的主要因素，而 LSTMs 是提升模型性能的一种方法。

Keras 中的 LSTM 层使得你指定输入序列的方向变得可能。可以通过设置 go backwards 参数为 True（默认的为 False）来完成。

具有反向输入序列的 Vanilla LSTM 模型的例子:

```python
model = Sequential()
model.add(LSTM(..., input_shape=(...), go_backwards=True))
```