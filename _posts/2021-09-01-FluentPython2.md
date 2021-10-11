---
layout: post
title: "流畅的python——序列构成的数组"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/09/1/
---

* Karmdown table of content
{:toc .toc}

# 内置序列概览

python标准库用C实现了丰富的序列类型，包括：

容器序列：list，tuple和collections.deque这些序列能够存储不同类型的数据
扁平序列：str，bytearry，bytearray，memoryview和array.array，这类容器只能容纳一种类型。

容器序列存储的是它们所包含的任意类型的对象的引用，而扁平序列里存放的是值而不是引用。

序列类型还可以按照能否被修改来分类。

可变序列：list，bytearray，array.array，collections.deque和memoryview
不可变序列：tuple，str和bytes

下图可以看到可变序列和不可变序列的一些类型。事实上，前者继承自后者。

![smiley](\assets\images\usedInBlogs\fluentpython\2-1.png)

关于可变序列与不可变序列的更多内容，可以再后续章节中慢慢领会。

# 列表推导和生成器表达式

## 列表推导

直接上代码

~~~python
symbols = '!@#$%^&*()'
codes = []
for symbol in symbols:
	codes.append(ord(symbol))
~~~~

下面两行代码等价于上面一段代码。
~~~python
symbols = '!@#$%^&*()'
codes = [ord(symbol) for symbol in symbols]
~~~~

这种写法就叫，列表推导式。不会的话一定要学一下。

有部分人会抨击列表推导的可读性。但是作为本教程的目标用户，这种程度的代码应该一眼就能看出其意义才对。所以本教程不对可读性做过多讨论。

*注：Python会忽略代码里[]、{}、()中的换行，因此如果你代码里有多行的列表、列表推导、生成器表达式、字典这一类的，可以省略不太好看的续行符\\*

*书中有用filter和map实现列表推导的讲解，算了，我不抄了，没啥意义*


## 生成器表达式

与列表推导式相比，生成器表达式背后遵守了迭代器协议，可以逐个的产出元素，而不是先建立一个完整的列表，然后再把这个列表传递到某个构造函数里。两者写法差不多，只不过生成器表达式把方括号换成了圆括号。

这一特性导致生成器表达式很适合作用在for循环中。

如以下代码：

~~~python
colors = ['black', 'white']
sizes = ['S', 'M', 'L']
for tshirt in ('%s %s' % (c, s) for c in colors for s in sizes):
	print(tshirt)
~~~

# 元组

元组除了可以作为不可变的列表之外，还可以用于没有字段名的记录。具名元组甚至可以作为key保持一致的字典来使用。

## 元组拆包

下面代码中的第一行、第三行都属于元组的拆包。

~~~python
city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
passport = ('USA', '31195855')
print('%s/%s' % passport)
~~~~

元组拆包可以应用到所有的可迭代对象上。唯一的要求是，被可迭代对象中的元素数量必须要根接受这些元素的元组的空挡数一致。除非我们使用'\*'来表示忽略多余的元素。

除了上面示例代码的拆包方式之外，下面这段代码的形式也是拆包

~~~python
b, a = a, b
~~~

也可以用\*把一个可迭代对象拆开作为函数的参数。

~~~python
#第一种写法
divmod(20, 8)
#第二种写法
t = (20, 8)
divmod(*t)
~~~

除此之外，在元组拆包中使用\*也可以使我们把注意力集中在袁祖德部分元素上。这应该算是一种经典的写法了。\*前缀只可以用在一个变量名前，但是这个变量的位置可以任意变换。

~~~python
a, b, *rest = range(5)
# a = 0, b = 1, rest = [2, 3, 4]
a, b, *rest = range(2)
# a = 0, b = 1, rest = []
a, *rest, b = range(3)
# a = 0, rest = [1], b = 2
~~~


## 嵌套拆包

直接上代码，自己品。

~~~python
metro_areas = ('Tokyo', 'JP', 36.933, (35.689722, 138.691667))
name, cc, pop, (latittude, longitude) = metro_areas
~~~~

## 具名元组

第一章其实已经用到了这种方式。


~~~python
from collections import namedtuple
Card = namedtuple('Card', ['rank', 'suit'])  # 第一章的声明方式

city = namedtuple('City', 'name country population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
print(str(tokyo.population))  # 36.933
print(str(totyo[1]))  # JP
~~~

创建一个具名元组需要两个参数，一个是类名，另一个是类的各个字段的名字。后者可以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串。

存放在对应字段里的数据要以一串参数的形式传入到构造函数中。

你可以通过字段名或是位置来访问一个字段的信息。

具名元组可以通过_asdict()方法将其转换为字典类型。

# 切片

s[a:b:c]表示对s在a和b中间以c为间隔取值。切片的表示中区间为左开右闭。

切片也可进行赋值。

~~~python
l = list(range(5))  # l = [0, 1, 2, 3, 4]

l[2:] = [0]  # l = [0, 1, 0]
~~~

# 对序列进行+和*

python程序员默认序列可以进行+和\*操作。两者都不会改变序列，而是构建一个全新的序列。

但是在a\*n这个语句中，如果a里的元素是可变元素的引用的话，就需要格外注意了。因为\*复制出来的元素可能是一个元素的多个引用，导致你如果更改其中一个元素的话也会同时更改其他元素。

例如：
~~~python
board = [['_'] * 2 for i in range(2)]  # board = 【['_', '_']， ['_', '_']】
board[0][1] = 'x' # board = 【['_', 'x']， ['_', '_']】

weird_board = [['_'] * 2] * 2  # weird_board = 【['_', '_']， ['_', '_']】
weird_board[0][1] = 'x'  # weird_board = 【['_', 'x']， ['_', 'x']】
~~~

# 其他

## 序列的增量赋值

a += b的背后是调用了a._iadd__(b)方法，如果没有实现该方法，则会退一步调用__add__，变成了a = a + b。

## list.sort 和 sorted

list.sort 会就地排序列表，sorted会新建一个列表作为返回值。sorted可以接受任何形式的可迭代对象作为参数。

## bisect

相当于为了方便二分搜索内置的数据结构。需要使用的时候自己查文档吧。

## deque

colletions.deque是线程安全的。


