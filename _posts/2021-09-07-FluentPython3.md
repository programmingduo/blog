---
layout: post
title: "流畅的python——字典和集合"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/09/7/
---

* Karmdown table of content
{:toc .toc}

# 范映射类型

映射类似于C++里面的map，可以理解为键值对。只有可散列的数据类型才可以作为这些映射里的键。

> 可散列：如果一个对象是可散列的，那么在这个对象的生命周期里，他的散列值是不变的。而且这个对象需要实现__hash__()方法。另外，可散列对象还要有__eq__()方法，这样才能跟其他键对比。如果两个可散列对象是相等的，那么他们的散列至一定是一样的。

字典声明：
~~~python
a = dict(one=1, two=2, three=3)
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
d = dict({'one': 1, 'two': 2, 'three': 3})

a == b == c == d  # true
~~~

# 字典推导

字典推导和列表推导大同小异，直接看代码：

~~~python
DIAL_CODES= [
	(86, 'Chine'),
	(91, 'India'),
	(1, 'United States'),
	(62, 'Indonesia'),
	(55, 'Brazil'),
	(92, 'Pakistan'),
	(880, 'Bangladesh'),
	(234, 'Nigeria'),
	(7, 'Russia'),
	(81, 'Japan')
]

country_code = {country: code for code, country in DIAL_CODES}
special_countries = {code: country.uper() for country, code in country_code.items()}
~~~

# 找不到的键

字典处理找不到的键，除了自己写ifelse之外，可以有三种处理：使用get()；使用setdefault()；使用defaultdict代替dict

个人推荐defaultdict和get()方法。如果很少会出现找不到的键，可以临时使用一下get()方法，否则还是建议声明成defaultdict。get()方法在找不到键的时候返回None，而defaultdict则会返回对象声明时的默认值。

所有的范映射类型在处理找不到的键的时候，都会牵扯到__missing__方法。而__missing__方法只会被__getitem__方法调用，对get或__contains__并没有影响。

# 字典的变种

collection模块中有一些宝藏类。

## collections.OrderedDict

这个类型在添加键的时候会保持顺序，因此键的迭代次序总是一致的。OrderedDIct的popitem方法默认删除并返回的是字典里的最后一个元素。但是如果将popittem的参数设置为last=False，则会删除第一个被添加进去的键。

## collections.Counter

究极好用的计数器。可以对可散列表对象进行计数。Counter实现了+和-对集合进行合并。most_common([n])会按照次序返回映射里最常见的n个元素。

~~~python
ct = collections.Counter('asecss')  # ct: Counter({'a':1, 's':3, 'e':1, 'c':1})
ct.update('ab')  # ct:Counter({'a':2, 's':3, 'e':1, 'c':1, 'b':1})
ct.most_common(2)  # [('s', 3), ('a', 2)]
~~~~

# 不可变映射类型

types 模块中引入了MappingProxyType，想要改变这个映射中的键值对的时候比较麻烦。需要使用的时候去查一下就流行了。

# 集合论

set。python内置类型，实现了-&|来计算两个集合的差集、交集、并集。

构造方法如下：

~~~python
a = {1, 2, 3}
a = set([1, 2 3])
~~~

第一种方法比第二种更快且更易懂。

集合推导：

~~~python
from unicodeddata import name
{char(i) for i in range(32, 256) if 'SIGN' in name(chr(i), '')}
~~~

# dict和set的背后

这一小节是本章最没用的了吧。但是是我最想写的。他稍微偏底层有点，不过学了会更清楚dict和set的一些特性。

## 效率

直接看表吧
![smiley](\assets\images\usedInBlogs\fluentpython\3-1.png)

反正意思就是，快，set快，dict也快。

## 字典中的散列表

散列表是指一个稀疏数组（总是有空白元素的数组成为稀疏数组）。散列表中的单元称为表元。每个键值对都占用一个表元，每个表元都有两个部分：对键的引用和对值的引用。

因为Python会保证有三分之一的表元是空的，所以在快要达到这个预知的时候，原油的散列表会被复制到一个更大的空间里面。如果要把一个对象放入散列表，首先要计算这个元素键的散列值。Python用hash()方法来实现这一点。

### 散列值和相等性

hash()实际调用的是__hash__方法，他要求如果两个值是相等的，那么他们的散列值必须也是相等的。

### 散列表算法

为了获取my_dict[search_key]，Python首先会调用hash(search_key)来计算其散列值，把这个值最低的几位数字当做偏移量，去散列表中查找表元（具体取几位取决于散列表的大小）。如果找到的表元是空的，则抛出KeyError异常。如果不是空的，则表元里会有一对fonud_key:found_value。如果found_key和search_key相等的话，就返回。如果不等说明发生了哈希冲突，按照预先定义好的冲突处理方式进一步寻找。直到找到空表元返回KeyError或找到search_key返回结果。

### dict的实现及其导致的结果

1. 键必须是可散列的

一个可散列的对象必须满足以下要求

(1) 支持hash()函数，并且通过__hash__()方法所得到的散列值是不变的。
(2）支持通过__eq__()方法来检测相等性
(3) 若a==b，则hash(a)==hash(b)也为真

2. 字典在内存上开销巨大

由于字典使用了散列表，散列表有必须是稀疏的，这导致它在空间上的效率低下。因此最好不要根据JSON风格，用由字典组成的列表来存放这些记录。

用元组替代字典就能节省空间的原因有两个：避免了散列表所耗费的空间；无须把记录中的字段的名字在每个元素里都存一遍。

3. 键查询很快

本质是哈希，用空间换时间。

4. 键的次序取决于添加顺序

如果发生冲突，那么添加顺序决定键的次序。

5. 往字典里添加新的元素会改变已有键的次序

为了保证散列表的稀疏性，有时添加新元素需要进行扩容。这一过程中可能会发生新的冲突，导致键的次序发生变化。

因此，千万不要同时对字典进行遍历和修改。Python3中的.keys()、.items()和.values()方法返回的都是字典视图。也就是说，这些方法返回的值更像是集合，而不是python2那样返回列表。视图还有动态性，他们可以实时反馈字典的变化。

## set的实现以及导致的结果

set和dict差不多。早期没有set的时候，都是给字典的键加上没有意义的值来取代set的。

set具有以下特点：

> 集合里的元素都必须是可散列的
> 集合很消耗内存
> 可以很搞笑的判断元素是否存在与某个集合
> 元素的次序取决于被添加的次序
> 往集合里添加元素可能会改变及合理已有元素的次序

