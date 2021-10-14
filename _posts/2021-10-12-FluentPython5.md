---
layout: post
title: "流畅的python——一等函数"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/10/12/
---

* Karmdown table of content
{:toc .toc}

在Python中，所有的函数都是一等对象。所谓一等对象，是指满足以下条件的程序实体：

> 在运行时创建
> 能赋值给变量或数据结构中的元素
> 能作为参数传递给函数
> 能作为函数的返回结果

# 把函数视作对象

~~~python
def factorial(n):
	""" return n! """
	return 1 if n < 2 else n * factorial(n - 1)

factorial.__doc__  # return n!
type(factorial)  # <class 'funciton'>
~~~~
\_\_doc\_\_是函数对象众多属性中的一个。factorial是function类的实例。

此外，我们也可以将factorial赋值给变量

~~~python
fact = factorial
fact(3)  # 6
~~~~

# 高阶函数


接收函数为参数，或者吧函数作为返回值的函数就是高阶函数。例如内置函数sorted，接受的参数key是一个函数，因此sorted便是一个高阶函数。
~~~python
fruits = ['apple', 'cherry', 'respberry']
sorted(fruits, key=len)
~~~

# 匿名函数

lambda关键词用于创建匿名函数。但是lambda仅能使用纯表达式，即不能进行赋值、不能使用while、try等python语句。

实例：

~~~python
g = lambda x : x**2
print g(4)

sorted(fruits, lambda word: word[::-1])
~~~~

# 可调用对象

python中包含的可调用对象分为以下7种：

> 用户定义的函数：使用def或lambda表达式创建
> 内置函数：使用C语言实现的函数，如len或time.strftime
> 内置方法：使用C语言实现的方法，如 dict.get
> 方法：在类的定义体中定义的函数
> 类：调用类时会运行类的__new__方法创建一个实例，然后运行__init__方法，初始化实例
> 类的实例：如果定义了__call__方法，那么它的实例可以作为函数调用
> 生成器函数：使用yield关键字的函数或方法，那么它的实例可以作为函数调用

类和函数逇区别见博客：https://www.cnblogs.com/mayugang/p/9977914.html

# 用户定义的可调用对象

类实现了__call__方法后它的实例就可以调用了。见示例

~~~python
import random

class BingoCage:
	def __init__(self, items):
		self._items = list(items)
		random.shuffle(self._items)

	def pick(self):
		# 应该使用try except的，但是我懒得敲了
		return self._items.pop()

	def __call__(self):
		return self.pick()

bingo = BingoCage(range(3))
bingo.pick()  # 1
bingo()   # 0
~~~~

# 从定位参数到仅限关键字参数

参数最好的特性之一就是实现了灵活的参数处理机制。调用函数时使用\*和\*\*“展开迭代对象”，映射到单个函数。

~~~python
def tag(name, *content, cls=None, **attrs):
	if cls is not None:
		attrs['cls'] = cls

	if attrs:
		attr_str = ''.join('%s="%s"' %(attr, value)
			for attr, value in sorted(attr.items()))
	else:
		attr_str = ''

	if content:
		return '\n'.join('<%s%s>%s</%s>' % (name, attr_str, c, name) for c in content)
	else:
		return '<%s%s />' % (name, attr_str)
~~~~

tag调用方式有很多。懒得码字了，直接如图所示吧。

![smiley](\assets\images\usedInBlogs\fluentpython\5-1.png)
![smiley](\assets\images\usedInBlogs\fluentpython\5-2.png)


> 传入单个定位参数，生成一个指定名称的空标签。
> 第一个参数后面的任意个参数会被\*content捕获，存入一个元组。
> tag函数签名中没有明确指定名称的关键字参数会被\*\*attrs捕获，存入一个字典。
> cls参数只能作为关键字参数传入。
> 调用tag函数时，即便第一个定位参数也能作为关键字参数传入。
> 在my_tag前面加上\*\*，字典中的所有元素作为单个参数传入，同名键会绑定到对应的具名参数上，余下的则被\*\*attrs捕获。

# 函数注解

这玩意就跟注释似的，python解释器也不会按照这个对变量进行检查，就是给程序员自己看的，“诶你看，这个参数是int，返回值是str”。程序员可以调用__annotations__进行查看。

~~~python
def clip(text: str, max_len:'int > 0'=90) -> str:
	pass
~~~~

