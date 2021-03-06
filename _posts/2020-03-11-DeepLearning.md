---
layout: post
title: "深度学习"
description: ""
categories: [DL]
tags: []
redirect_from:
  - /2020/03/11/
---

* Karmdown table of content
{:toc .toc}

# 优化方法

参考博客：[神经网络中的各种优化方法](https://blog.csdn.net/autocyz/article/details/83114245)

## Gradient Descent

![smiley](\assets\images\usedInBlogs\DL\0.png)

上面的公式就是梯度下降法的基本形式，其中 η 是学习率 ΔJ(θ)是损失函数 J(θ) 关于模型参数 θ 的梯度。

因为这里的损失函数是在整个数据集上进行计算得到的均值，所以每更新一次模型参数，就要对整个数据集进行一个计算，可想而知这样非常的慢，并且当数据集变得非常大的时候，如此多的数据没法都load到内存中。

特点：

对于凸目标函数，肯定可以找到全局最优，对于非凸目标函数，可以找到局部最优

每次更新都对整个数据集进行计算，计算量大

## Stochastic Gradient Descent

![smiley](\assets\images\usedInBlogs\DL\1.png)

随机梯度下降法和梯度下降法其实是走的两个极端，梯度下降法是每次更新都计算整个数据集的loss，而随机梯度下降法每次更新都只用了一对样本，即上面公式中的一对样本(xi,yi)。由于每个样本都会对模型进行更新，所以模型更新的特别频繁，参数就会变成高方差，损失函数的波动也会有很大强度的变化。

特点：

每次只用一个样本进行更新，计算量小，更新频率高

容易导致模型超调不稳定，收敛也不稳定

## Mini Batch Gradient Descent

![smiley](\assets\images\usedInBlogs\DL\3.png)

其中B就是自己设定的batch size的大小。

特点：
每次更新模型时，采用一部分数据进行计算
现在几乎在所有的深度学习应用中，都使用的是mini batch梯度下降法
正是因为使用广泛，所以很多深度学习的库和框架都对这方面的矩阵操作进行了特别优化

问题：

1、很难选择一个合适学习率，需要我们大量的尝试。太小的学习率会导致模型收敛变得缓慢，导致训练时间变长；太大的学习率会阻碍模型的收敛，使损失函数在最优值附近来回波动甚至是发散。

2、对于所有的参数，均使用相同的学习率。如果有的参数的梯度很大，有的很小，使用相同的学习率显然就不是很合适。

3、在非凸函数的优化过程中，我们往往希望模型能够跳过那些局部极值点，去找一个更好的极值。在实际训练过程中，鞍点也是非常难以解决的问题。

## Momentum

![smiley](\assets\images\usedInBlogs\DL\4.png)

从上述公式（1）可以看出，当当前的梯度方向（ΔJ(θ)的正负号）和 V(t−1)的方向相同时，V(t)>ηΔJ(θ) 所以参数 θ 的变化幅度会增大，从而加快梯度下降法的幅度；而当方向不同时，会逐步减小当前更新的幅度。这样可以有效的对梯度下降法进行加速，同时提高模型的稳定性。

## Nesterov

![smiley](\assets\images\usedInBlogs\DL\5.png)

特点：

能够在对梯度下降法进行加速的同时，提高稳定性，避免越过极值

## Adagrad

根据每个参数以前的梯度情况，对不同参数使用不同的学习率，同时动态调整参数学习率

![smiley](\assets\images\usedInBlogs\DL\6.png)

## AdaDelta

## Adam

## 总结

Adam works well in practice and outperforms other Adaptive techniques.



# 损失函数

## 可能损失负对数

~~~python
torch.nn.functional.nll_loss(input, target, weight=None, size_average=True)
~~~~

input - (N,C) C=类别的个数或者在例子2D-loss中的(N, C, H, W)
target - (N) 其大小是 0 <= targets[i] <= C-1 1. 
weight (Variable, optional) – 每个类都有一个手动的重新分配的重量。如果给定，则必须是一个大小为“nclasses”的变量
size_average (bool, optional) – 默认情况下，是mini-batchloss的平均值；如果size_average=False，则是mini-batchloss的总和。
ignore_index (int, optional) - 指定一个被忽略的目标值，并且不影响输入梯度。当size平均值为真时，在非被忽略的目标上的损失是平均的。


# 梯度问题

## sigmoid

缺点：
1. sigmoid更容易引起梯度消失的问题。原因有两个：sigmoid导数小于0.25，反向传播过程中容易导致梯度相乘结果趋于0；sigmoid在x>5的时候导数已经趋于0了。
2. sigmoid函数输出不是“零为中心”(zero-centered）。一个多层的sigmoid神经网络，如果你的输入x都是正数，那么在反向传播中w的梯度传播到网络的某一处时，权值的变化是要么全正要么全负。
3. 指数函数的计算是比较消耗计算资源的。

## tanh

优点：
解决了sigmoid的输出非“零为中心”的问题。

缺点：
1. 依然有sigmoid函数过饱和的问题。
2. 依然指数运算。

## relu

优点：
1. ReLU解决了梯度消失的问题，至少x在正区间内，神经元不会饱和。
2. 由于ReLU线性、非饱和的形式，在SGD中能够快速收敛。
3. 计算速度要快很多。ReLU函数只有线性关系，不需要指数计算，不管在前向传播还是反向传播，计算速度都比sigmoid和tanh快。

缺点：
1. ReLU的输出不是“零为中心”(Notzero-centered output)。
2. 随着训练的进行，可能会出现神经元死亡，权重无法更新的情况。这种神经元的死亡是不可逆转的死亡。



# 过拟合解决方法

1. 增加数据量
2. L正则式
	cost = (Wx - realY)^2 + abs(W)
	cost = (Wx - realY)^2 + (W)^2
3. dropout regularization
	神经网络中随机忽略一些节点


# 加速方法：

## momentum：

惯性原则

## AdaGrad

每个参数都有自己的学习率

## RMSProp

Momentum+AdaGrad

## Adam

# RNN

代码还没调通，训练虽然参数更新了但是效果没有提升

~~~python
###model

class RNN(nn.Module):
	def __init__(self, hidden_size, sen_length, out_size, batch_size = 1):
		super().__init__()
		self.batch_size = batch_size
		self.hidden_size = hidden_size
		self.hidden = self.init_hidden()
		self.i2o = nn.Linear(hidden_size + sen_length, out_size)
		self.i2h = nn.Linear(hidden_size + sen_length, hidden_size)
		# self.softmax = F.log_softmax(dim = 1)

	def forward(self, input):
		# print(input)
		# print(self.hidden)
		# print(input.size())
		# print(self.hidden.size())
		combined = torch.cat((input.float(), self.hidden), 1)
		self.hidden = self.i2h(combined)
		output = self.i2o(combined)
		output = F.log_softmax(output, dim = -1)
		return output

	def init_hidden(self):
		return Variable(torch.zeros(self.batch_size, self.hidden_size), requires_grad=True)

~~~~


~~~python
# -*- encoding:utf-8 -*-
from torchtext import data
from torchtext import datasets
import torch.nn as nn
from torch import optim
import torch.nn.functional as F
from torchtext.vocab import GloVe
from model import *

sen_length = 20
BATCH_SIZE = 32
TEXT = data.Field(lower=True, batch_first=True, fix_length=sen_length)
LABEL = data.Field(sequential=False)
train, test = datasets.IMDB.splits(TEXT, LABEL)

TEXT.build_vocab(train, vectors=GloVe(name="6B", dim=50), max_size=10000, min_freq=10)
LABEL.build_vocab(train)

# print(len(train))
# print(vars(train[0]))
# print(train[0].text)
# print(test[0])

def fit(model, data_loader, phase='training', volatile=False, is_cuda=False):
	if phase == 'training':
		model.train()
	if phase == 'validation':
		model.eval()
		volatile = True

	running_loss = 0.0
	running_correct = 0
	optimizer = optim.Adam(model.parameters(), lr=1e-3)
	for batch_idx, batch in enumerate(data_loader):
		# for param in model.named_parameters():
		# 	print(param)
		if batch.label.size(0) != BATCH_SIZE:
			continue
		txt, target = batch.text, batch.label
		if is_cuda:
			txt, target = txt.cude(), target.cuda()
		if phase == 'training':
			optimizer.zero_grad()

		output = model(txt)
		loss = F.nll_loss(output, target)
		#可能损失负对数
		running_loss += F.nll_loss(output, target, size_average=False).data.item()
		# print(running_loss)
		#????
		preds = output.data.max(dim = 1, keepdim= True)[1]
		running_correct += preds.eq(target.data.view_as(preds)).cpu().sum()
		if phase == 'training':
			loss.backward(retain_graph=True)
			optimizer.step()

	loss = running_loss / len(data_loader.dataset)
	accuracy = 100.0 * running_correct / len(data_loader.dataset)
	# print('epoch: %d, loss: %f, accuracy: %f' %(epoch, loss, accuracy))
	return loss, accuracy

# model = EmbNet(10000, 20, 50)
model = RNN(32, sen_length, 3, batch_size = BATCH_SIZE)
train_iter, test_iter = data.BucketIterator.splits((train, test), batch_size = BATCH_SIZE, device = -1, shuffle = True)
train_iter.repeat = False
test_iter.repeat = False##

for epoch in range(1, 50):
	epoch_loss, epoch_accuracy = fit(model, train_iter, phase='training')
	print('epoch: %d, loss: %f, accuracy: %f' %(epoch, epoch_loss, epoch_accuracy))
	val_epoch_loss, val_epoch_accuracy = fit(model, test_iter, phase = 'validation')
	print('val epoch: %d, loss: %f, accuracy: %f' %(epoch, val_epoch_loss, val_epoch_accuracy))

~~~~~



# LSTM

参考博文：[理解 LSTM(Long Short-Term Memory, LSTM) 网络](https://www.cnblogs.com/wangduo/p/6773601.html)

实际上有一篇英文的博客很好，看了一些翻译自这个博客的文章质量都还挺高的。博客原文：[Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)

# DBN(Deep Believe Nets)

既可用于非监督学习，类似于一个自编码机；也可用于监督学习，作为分类器使用

DBN的组成元素是受限玻尔兹曼机(RBM)

## RBM

RBM是一个概率生成模型，目的在于建立一个观察数据和标签之间的联合分布。

常见的RBM是二值的，即不管是隐层还是显层都只能取值为0或1。

RBM有两层神经元组成：显层和隐层。显层有显元组成，用于输入训练数据。隐层有隐元组成，用于特征检测器。

RBM通过一种特殊的深度学习方法来表示输入数据。具体来讲，先将输入数据（v0）（显层）映射到隐层（h0），在由隐层的神经元（h0）映射回显层（v1）。如果输入数据（v0）和映射回的显层（v1）结果一样，则可以认为隐层可以作为输入数据的另一种表达，从而达到了特征提取的过程。

## 贝叶斯信念网络(贝叶斯网络)

贝叶斯网络是一种邮向无环图模型，是一种概率图模型。

在朴素贝叶斯网络的基础上加上了随机变量之间有单向依赖关系的假设。因此需要在贝叶斯网络基础上新增一步构造随机变量之间的拓扑关系的步骤。


## DBN

如果我们将隐层的层数增加，则得到了Deep Boltzmann Machine(DBM)。如果我们在靠近可视层的部分使用贝叶斯信念网络（即有向图模型，这里仍然限制层内节点之间么有链接），而在最远离可视层的部分使用RBM，则得到了DBN

典型的网络如图所示

![smiley](\assets\images\usedInBlogs\DL\7.png)