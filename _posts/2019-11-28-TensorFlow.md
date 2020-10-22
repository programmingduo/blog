---
layout: post
title: "TensorFlow"
description: "TensorFlow learning"
categories: [nlp]
tags: [encoding model]
redirect_from:
  - /2019/11/28/
---

* Karmdown table of content
{:toc .toc}

# 简介

TensorFlow 是由 Google Brain 团队为深度神经网络（DNN）开发的功能强大的开源软件库，于 2015 年 11 月首次发布。开源深度学习库 TensorFlow 允许将深度神经网络的计算部署到任意数量的 CPU 或 GPU 的服务器、PC 或移动设备上，且只利用一个 TensorFlow API。

TensorFlow 则还有更多的特点，如下：
	支持所有流行语言，如 Python、C++、Java、R和Go。
	可以在多种平台上工作，甚至是移动平台和分布式平台。
	它受到所有云服务（AWS、Google和Azure）的支持。
	Keras——高级神经网络 API，已经与 TensorFlow 整合。
	与 Torch/Theano 比较，TensorFlow 拥有更好的计算图表可视化。
	允许模型部署到工业生产中，并且容易使用。
	有非常好的社区支持。
	TensorFlow 不仅仅是一个软件库，它是一套包括 TensorFlow，TensorBoard 和 TensorServing 的软件。

TensorFlow 成为最受欢迎的深度学习库，原因如下：
	TensorFlow 是一个强大的库，用于执行大规模的数值计算，如矩阵乘法或自动微分。这两个计算是实现和训练 DNN 所必需的。
	TensorFlow 在后端使用 C/C++，这使得计算速度更快。
	TensorFlow 有一个高级机器学习 API（tf.contrib.learn），可以更容易地配置、训练和评估大量的机器学习模型。
	可以在 TensorFlow 上使用高级深度学习库 Keras。Keras 非常便于用户使用，并且可以轻松快速地进行原型设计。它支持各种 DNN，如RNN、CNN，甚至是两者的组合。

# 常量、变量和占位符

## 常量声明

声明一个标量常量：

~~~py
t_1 = tf.constant(4)
~~~~

一个形如 [1，3] 的常量向量：

~~~py
t_2 = tf.constant([4,3,2])
~~~~

创建一个所有元素为零的张量:
tf.zeros([M,N],tf.dtype)

~~~py
zero_t = tf.zeros([2,3],tf.int32)
# Results in an 2x3 array of zeros:[[0 0 0],[0 0 0]]
~~~~

创建一个所有元素都设为 1 的张量
tf.ones([M,N],tf,dtype)

~~~py
ones_t = tf.ones([2,3],tf.int32)
# Results in an 2x3 array of ones:[[1 1 1],[1 1 1]]
~~~



还可以创建与现有 Numpy 数组或张量常量具有相同形状的张量常量，如下所示：

~~~py
tf.zeros_like(t_2)
# Create a zero matrix of same shape as t_2
tf.ones_like(t_2)
# create a ones matrix of same shape as t_2
~~~~

在一定范围内生成一个从初值到终值等差排布的序列：
tf.linspace(start,stop,num)

相应的值为 (stop-start)/(num-1)。例如：

~~~py
range_t = tf.linspace(2.0,5.0,5)
#We get:[2. 2.75 3.5 4.25 5.]
~~~~~

从开始（默认值=0）生成一个数字序列，增量为 delta（默认值=1），直到终值（但不包括终值）：
tf.range(start,limit,delta)

~~~py
range_t = tf.range(10)
#Result:[0 1 2 3 4 5 6 7 8 9]
~~~~

TensorFlow 允许创建具有不同分布的随机张量：
	
	使用以下语句创建一个具有一定均值（默认值=0.0）和标准差（默认值=1.0）、形状为 [M，N] 的正态分布随机数组：
	t_random = tf.random_normal([2, 3], mean = 2.0, stddev = 4, seed = 12)

	创建一个具有一定均值（默认值=0.0）和标准差（默认值=1.0）、形状为 [M，N] 的截尾正态分布随机数组：
	t_random = tf.truncated_normal([1, 5], stddev = 2, seed = 12)

	要在种子的 [minval（default=0），maxval] 范围内创建形状为 [M，N] 的给定伽马分布随机数组，请执行如下语句：
	t_random = tf.random_uniform([2, 3], maxval = 4, seed = 12)

	要将给定的张量随机裁剪为指定的大小，使用以下语句：
	tf.random_crop(t_random, [2, 5], seed = 12)
	这将导致随机从张量 t_random 中裁剪出一个大小为 [2，5] 的张量。

	沿着它的第一维随机排列张量
	tf.random_shuffle(t_random)
	# t_random 是想要重新排序的张量

	随机生成的张量受初始种子值的影响。要在多次运行或会话中获得相同的随机数，应该将种子设置为一个常数值。当使用大量的随机张量时，可以使用 tf.set_random_seed() 来为所有随机产生的张量设置种子。以下命令将所有会话的随机张量的种子设置为 54：
	tf.set_random_seed(54)

## 变量声明

变量通过使用变量类来创建。变量的定义还包括应该初始化的常量/随机值。
下面的代码中创建了两个不同的张量变量 t_a 和 t_b。两者将被初始化为形状为 [50，50] 的随机均匀分布，最小值=0，最大值=10：

~~~py
rand_t = tf.random_uniform([50, 50], 0, 10, seed = 0)
t_a = tf.Variable(rand_t)
t_b = tf.Variable(rand_t)

weights = tf.Variable(tf.random_normal([100, 100], stddev = 21))
bias = tf.Variable(tf.zeros[100], name = 'biases')
weight2 = tf.Variable(weights.initialized_value(), name = 'w2')
~~~~

保存变量：使用 Saver 类来保存变量，定义一个 Saver 操作对象：
saver = tf.train.Saver()

## 占位符声明

占位符用于将数据提供给计算图使用feed_dict输入数据。

tf.placeholder(dtype, shape = None, name = None)

示例代码：

~~~py
x = tf.placeholder("float")
y = 2 * x
data = tf.random_uniform([4, 5], 10)
with tf.Session() as sess:
	x_data = sess.run(data)
	print(sess.run(y, feed_dict = {x: x_data}))
~~~~

## 解读分析

需要注意的是，所有常量、变量和占位符将在代码的计算图部分中定义。如果在定义部分使用 print 语句，只会得到有关张量类型的信息，而不是它的值。

为了得到相关的值，需要创建会话图并对需要提取的张量显式使用运行命令，如下所示：
print(sess.run(t_1))
#Will print the value of t_1 defined in step 1

很多时候需要大规模的常量张量对象；在这种情况下，为了优化内存，最好将它们声明为一个可训练标志设置为 False 的变量：
t_large = tf.Varible(large_array,trainable = False)

# 矩阵基本操作

~~~python
sess = tf.InteractiveSession()

#define a 5*5 Identity matrix
I_matrix = tf.eye(5)
print(I_matrix.eval())

#define a Variable intialized to a 10*10 identity matrix
X = tf.Variable(tf.eye(10))
X.initializer.run()
print(X.eval())

A = tf.Variable(tf.random.normal([5, 10]))
A.initializer.run()
print(A.eval())

product = tf.matmul(A, X)
print(product.eval())
b = tf.Variable(tf.random.uniform([5, 10], 0, 2, dtype = tf.int32))
b.initializer.run()
# cast to float data type
b_new = tf.cast(b, dtype = tf.float32)

t_sum = tf.add(product, b_new)
t_sub = product - b_new

~~~~

一些其他有用的矩阵操作，如按元素相乘、乘以一个标量、按元素相除、按元素余数相除等，可以执行如下语句：

~~~python
a = tf.Variable(tf.random.uniform([2, 3], 1, 9, dtype = tf.float32))
b = tf.Variable(tf.random.uniform([2, 3], 1, 10, dtype = tf.float32))

#element in same position do multiplication
#element-wise Multiplication
A = a * b

#Multiplication with a scalar 2
B = tf.scalar_mul(2, A)

#element-wise division
C = tf.div(a, b)

#element-wise remainder of division
D = tf.mod(a, b)

init_op = tf.global_variables_initializer()
with tf.Session() as sess:
	sess.run(init_op)
	writer = tf.summary.FileWriter('graphs', sess.graph)
	a, b, A_R, B_R, C_R, D_R = sess.run([a, b, A, B, C, D])
	print(a, b, A_R, B_R, C_R, D_R)

writer.close()

~~~~

tf.div 返回的张量的类型与第一个参数类型一致

## 解读分析

所有加法、减、除、乘（按元素相乘）、取余等矩阵的算术运算都要求两个张量矩阵是相同的数据类型，否则就会产生错误。可以使用 tf.cast() 将张量从一种数据类型转换为另一种数据类型。

如果在整数张量之间进行除法，最好使用 tf.truediv(a，b)，因为它首先将整数张量转换为浮点类，然后再执行按位相除。

# 数据流图可视化

http://c.biancheng.net/view/1887.html


# random 常用函数介绍

## 正态随机分布

rf.random_normal

tf.random_normal(shape,mean=0.0,stddev=1.0,dtype=tf.float32,seed=None,name=None)

shape定义维度，mean定义均值，stddev定义方差，dtype定义类型，seed定义种子，name定义名称

## 标准正态分布

tf.truncated_normal 

tf.truncated_normal(shape, mean=0.0, stddev=1.0, dtype=tf.float32, seed=None, name=None)

shape定义维度，mean定义均值，stddev定义方差，dtype定义类型，seed定义种子，name定义名称

从截断的正态分布中输出随机值。 如果随机出来的值偏离平均值超过2个标准差，那么这个数将会被重新随机。

## 均匀分布

tf.random_uniform(shape, minval=0.0, maxval=1.0, dtype=tf.float32, seed=None, name=None)

shape定义维度，minval区间最小值，maxval区间最大值，dtype定义类型，seed定义种子，name定义名称

## 随机的交换位置 

tf.random_shuffle(value, seed=None, name=None) 

 value是一个给定的张量，seed定义的种子，name定义名称

c = tf.constant([[1,2],[3,4],[5,6]])
shuff = tf.random_shuffle(value=c,seed=1,name="shuff")
with tf.Session() as sess:
    print (sess.run(shuff))

输出：
[[1 2] 
[5 6] 
[3 4]]

## 设置种子 

tf.set_random_seed(seed)  
seed是给定的种子