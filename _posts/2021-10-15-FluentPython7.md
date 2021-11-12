---
layout: post
title: "流畅的python——函数装饰器和闭包"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/10/15/
---

* Karmdown table of content
{:toc .toc}

python学习过程中，从装饰器开始就逐渐牛逼了。但是在日常的开发过程中，如果不刻意使用的话，可能很难学会这块，更别提体会到其中的精髓了。所以建议写代码过程中刻意使用一下。慢慢体会其中的奥秘。

# 装饰器基础知识

装饰器是可调用的对象，器参数是另一个函数。装饰器可能会处理被装饰的函数，然后将其返回，或者把他替换成另一个函数或可调用对象。

基本语法：

~~~python
@decorate
def target():
	pass
~~~

上述语句相当于：

~~~python
def target():
	pass

target = decorate(target)
~~~

来看一个例子吧

![smiley](\assets\images\usedInBlogs\fluentpython\7-1.png)

# python何时执行装饰器

装饰器在函数定义之后立即运行，这通常是在导入的时候执行（即python加载模块时）。还是看个例子。

![smiley](\assets\images\usedInBlogs\fluentpython\7-2.png)
![smiley](\assets\images\usedInBlogs\fluentpython\7-3.png)

注意，register在模块中其他函数之前运行（两次）。调用register时，传给它的参数是被装饰的函数，例如<function f1 at 0x100631bf8>。加载模块后，registry中有两个被装饰函数的引用：f1和f2。这两个函数，以及f3，只在main明确调用它们时才执行。

# 使用装饰器改进“策略”模式

直接上代码

~~~python
promos = []

def promotion(prom_func):
	promos.append(prom_func)
	return prom_func

@promotion
def fidelity(order):
	# 为积分1000分以上的提供5折优惠
	return order.total * 0.5 if order.customer.fidelity >= 1000 else 0

@promotion
def bulk_item(order):
	# 单个商品数量在20以上提供1折优惠
	discount = 0
	for item in order.cart:
		if item.quantity >= 20:
			discount += item.total * 0.1

def best_promo(order):
	return max(promo(order) for promo in promos)
~~~

# 变量作用域原则

这里有一个小插曲，感觉不属于这一章，但是是python一个比较重要的知识。

python不要求声明变量，但是会把函数定义体中赋值的变量认作是局部变量。

看例子。

![smiley](\assets\images\usedInBlogs\fluentpython\7-4.png)

如果把b定义为全局变量，就不会出错。

![smiley](\assets\images\usedInBlogs\fluentpython\7-5.png)

但是下面这段代码，就会出错。因为“python不要求声明变量，但是会把函数定义体中赋值的变量认作是局部变量。”

![smiley](\assets\images\usedInBlogs\fluentpython\7-6.png)

# 闭包

闭包指延伸了作用域的函数，其中包含函数定义体中引用、但是不在定义体中定义的非全局变量。

是不是很绕，看段代码。

假如有一个名为avg的函数，作用是计算不断增加的序列值的平均数。假设实现过程中要存储所有计算过的数值。

一种朴素的想法是定义一个class，然后搞一个self.series对所有序列值进行存储，计算的时候求和取平均即可。但是这里看一种高阶函数的实现方法。

~~~python
def make_averager():
	series = []
	def averager(new_value):
		series.append(new_value)
		total = sum(series)
		return total / len(series)

avg = make_averager()
avg(9)  # 9
avg(10)  # 9.5
~~~

注意，series是make_averager函数的局部变量，因为那个函数的定义体中初始化了series：series = []。可是，调用avg(10)时，make_averager函数已经返回了，而它的本地作用域也一去不复返了。在averager函数中，series是自由变量（free variable）。这是一个技术术语，指未在本地作用域中绑定的变量。见下图。

![smiley](\assets\images\usedInBlogs\fluentpython\7-7.png)

综上，闭包是一种函数。它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用于不可用了，但是仍能使用那些绑定。

# nonlocal声明

来看看下面这段代码

![smiley](\assets\images\usedInBlogs\fluentpython\7-8.png)

运行结果：

![smiley](\assets\images\usedInBlogs\fluentpython\7-9.png)

当count是数字或者任何不可变类型的时候，count+=1会隐形的为其创建局部变量，导致count不在是自由变量了。为了解决这个问题，python3引入了nonlocal变量。她的作用是将变量标记为自由变量。看代码。

![smiley](\assets\images\usedInBlogs\fluentpython\7-10.png)

# 后续

原书中讲解了自己实现装饰器的方法，也卸了标准库常中用到的装饰器。但是这里感觉并没有使用过，而且也比较复杂，所以本博客中不进行抄录。所以讲来讲去算是做了科普。这么看来，这一章也没什么太大的作用。

本文开头说慢慢体会其中奥妙来着。好像骗了你（我先逃了）。