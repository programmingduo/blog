---
layout: post
title: "流畅的python——首章"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/08/31/
---

* Karmdown table of content
{:toc .toc}

# 前言

学习python已经两年多了。在此之前主要用的语言是C++，除此之外写过java、go、c#的项目，也做过php+js+html的网站。有过一定的语言基础之后在学习python的感受就是，语法根本不重要，写代码的过程中可以边写边搜，要写for循环，直接在百度框内输入，“python for”即可，连google都用不到。需要异常处理，就再输入“python 异常处理”，基本上都不用滚动页面，直接在第一条搜索结果中就有你想要的了。这么下来，一周左右，基本上也就会写python了。等到后面需要封装、需要写稍微大点的项目的时候，再搜一下python的文件import机制、类机制。说来说去，就是，百度是我们的好朋友，面向搜索编程就可以了。

但是这些仅仅在实现初步功能的时候奏效。如果想要优雅的编程，或者在项目变大的时候高效的编程，还需要进行进一步的学习。去学习python独有的一些语法功能，学习python的一些数据结构及其机制。为此，打算出一个python学习系列，**旨在拥有基本的语法基础条件下进一步提升python的编程能力。**

本系列大多数内容参考自《流畅的Python》。其实也不是参考人家啦，基本上是搬运过来的。当然，也会结合自己两年来在深度学习领域的实践，进行内容筛选并且输出一些自己的理解。

另外，提一点个人理解。学习的结果并不一定是你真的记住了多少东西。事实上，我们学过的东西大多数时候并不需要使用到他们，即使使用到也可能并不会多次使用。但是，在心理学中对于学习结果的检测除了直接让你输出自己的学习内容（如背诵、填空等），还可以衡量你再次学会（如可以拼写一个单词）这部分知识需要多少时间，或者给你若干选项你是否能够选出正确结果（就是给你一定提示你能不能提取到自己的知识）。**我们在学习编程语言的过程中，可能你学完之后并不能够把这部分完整的记住，但是你只要知道，他可以用什么技术手段实现即可，甚至你只要记住他可以实现，然后你知道该搜什么，便达到了学习目的。**

本教程只筛选了在深度学习项目中可能用到的知识，所以推荐非本领域的读者阅读原著。当然，深度学习领域的读者也可阅读原著，并且推荐这样做。**由于本教程针对的是已经有一定编程基础的读者，所以读者在目录中应当能够大概明白自己需要看哪些章节。**

# showoff

为了抓人眼球，首先秀一把。对于Python的初级学者，这段代码足以让人拍案称赞了。但是等你学完本系列之后就会觉得，这段代码仅仅是小儿科而已。**本部分的知识有利于写一些基础的数据结构类。在深度学习项目中，可以根据需求写非常灵活的dataloader**


## 示例1-1	一摞有序的纸牌
~~~python
import collections

Card = collections.namedtuple('Card', ['rank', 'suit'])

class FrenchDeck:
	ranks = [str(n) for n in range(2, 11)] + list('JQKA')
	suits = 'spades diamonds clubs hearts'.split()

	def __init__(self):
		self.cards = [Card(rank, suit) for rank in self.ranks\
									   for suit in self.suits]

	def __len__(self):
		return len(self.cards)

	def __getitem__(self, position):
		return self.cards[position]
~~~~

这一示例的主角 FrenchDeck，短小精悍。因为短短几行代码，就表示我们可以对这副扑克牌执行以下操作：

~~~python
from random import choice
deck = FrenchDeck

len(deck)  # 获取扑克牌的长度
deck[0]  # 获取第一张扑克牌
choice(deck)  #  随机抽取一张扑克牌
deck[:3]  # 获取前三张扑克牌
deck[12::13]  # 获取所有的A
for card in deck:  # 顺序遍历扑克牌
	print(card)
for card in reversed(deck):  # 逆序遍历扑克牌
	print(card)
Card('1', 'beats') in deck  # 判断某张牌是否在这摞扑克牌中
~~~~

这些方法中除了len函数之外，大多数功能都是通过实现\_\_getitem\_\_方法实现的。因为实现了\_\_getitem\_\_方法，这摞纸牌变成了可迭代的了。而在一个类型没有实现\_\_contain\_\_方法时，调用in运算符也会按顺序进行一遍迭代搜索。

这些类似的特殊方法，是由Python解释器调用的。我们并不会使用my_object.\_\_len__()这种调用方式，而是使用len(my_object)。具体哪种函数调用哪个特殊方法，则依靠Python解释器。通常并不会直接调用特殊方法，甚至调用特殊方法的频率远远低于实现他们的次数。唯一的例外可能是\_\_init\_\_方法，因为在继承过程中需要调用超类的构造器。

关于特殊方法的功能以及调用该方法的函数统一放到一张表格当中，在本系列的教程中，会逐渐完善这一表格。目前已经用到了\_\_len\_\_、 \_\_getitem\_\_方法。

## 示例1-2 一个简单的二维向量

~~~python
from math import hypot

class Vector:
	def __init__(self, x, y):
		self.x = x
		self.y = y

	def __repr__(self):
		return 'Vector(%r, %r)' % (self.x, self.y)

	def __abs__(self):
		return hypot(self.x, self.y)

	def __bool__(self):
		return bool(abs(self))

	def __add__(self, other):
		x = self.x + other.x
		y = self.y + other.y
		return Vecotr(x, y)

	def __mul__(self, scalar):
		return Vector(self.x * scalar, self.y * scalar)
~~~~

### 字符串表示形式

Python有一个内置函数repr，可以把一个对象用字符串的形式表达出来以便辨认。repr函数就是通过调用__repr__方法得到的。在使用%符号格式化字符串和str.format的方法中，都是通过__repr__实现的。

部分类会实现__str__函数。他们的区别在于，\_\_str\_\_只有在str()函数中才会使用，或是在print函数的调用过程中使用。而如果一个类没有实现__str__，则会调用__repr__。对此，我们可以理解为，\_\_repr\_\_更适合开发者自己使用，而\_\_str__则是更多的为比较业余的用户提供输出信息。


### 算术运算符

\_\_add\_\_和\_\_mul\_\_是为了实现+和\*操作。

### 布尔值

bool(x)背后调用的是x.__bool__()，如果没有实现这一函数则会调用x.__len__()。若返回0，则为False，否则为True。除此之外，默认情况下自己定义的类的实例总是被认为是真的。