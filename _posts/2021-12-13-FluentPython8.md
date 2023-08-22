---
layout: post
title: "流畅的python——对象引用、可变性和垃圾回收"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/12/13/
---

* Karmdown table of content
{:toc .toc}

# 变量不是盒子

python中变量是引用式变量，是先创立对象，之后把变量分配给对象。

# 标识、相等性和别名

别名：两个变量指向同一个对象，他们的id一致。如果通过一个变量改变了对象，另一个变量指向的对象也会被改变。代码：

![smiley](\assets\images\usedInBlogs\fluentpython\9-1.png)

但是如果另外声明了一个新的对象，即使对象的取值相同，他们id也不同。但是使用‘==’进行比较时，得到的是true。

![smiley](\assets\images\usedInBlogs\fluentpython\9-2.png)

ID一定是唯一的数值标识，在对象的生命周期内不会改变。‘==’比较的是对象的值（对象中保存的数据），is比较的是对象的标识。

is运算符比==速度快，因为它不能重载，所以Python不用寻找并调用特殊方法，而是直接比较两个整数ID。而a==b是语法糖，等同于a.__eq__(b)。继承自object的__eq__方法比较两个对象的ID，结果与is一样。但是多数内置类型使用更有意义的方式覆盖了__eq__方法，会考虑对象属性的值。

# 默认做浅复制

复制列表（或多数内置的可变集合）最简单的方式是使用内置的类型构造方法（如：L2=list(L1) 或 L2=L1[:]）。这二者使用的是浅复制（即复制了最外层的容器，副本中的元素是元容器中元素的引用）。如果所有元素都是不可变的，那么这个做法没有问题。但是如果有可变的元素，那么这么做就会导致意想不到的问题。

![smiley](\assets\images\usedInBlogs\fluentpython\9-3.png)

为了解决浅复制的问题，一般建议使用copy.deepcopy()（不是copy()方法，原书有对两者区别的描述部分）方法对可变对象进行复制。

# 函数参数作为引用时

Python唯一支持的参数传递模式是共享传参。共享传参指函数的各个姓氏参数获得实参中各个引用的副本。也就是说，函数内部的形参是实参的别名。

这种方案的结果就是，函数可能会修改作为参数传入的可变对象，但是无法修改哪些对象的标识（ID）。

## 不要使用可变类型作为参数的默认值

看实例吧。

![smiley](\assets\images\usedInBlogs\fluentpython\9-4.png)

然后就会出现这么一个离奇的bug。

![smiley](\assets\images\usedInBlogs\fluentpython\9-5.png)

默认值在定义函数时计算（通常在加载模块时），因此默认值变成了函数对象的属性。因此，如果默认值是可变对象，而且修改了它的值，那么后续的函数调用都会收到影响。

在使用参数过程中，要谨记API设计的“最少惊讶”原则。如果参数是可变参数，要注意是否希望函数内对参数进行的修改体现在函数外。

# del和垃圾回收

python中对象绝不会自行销毁。但是无法得到对象时，可能会被当做垃圾回收。

del语句删除名称，而不是对象。但是如果删除的变量保存的是对象的最后一个引用，或者无法得到对象时，del命令会导致对象被当做垃圾回收。

# 弱引用

正是因为有引用，对象才会在内存中存在。当对象的引用数量归零后，垃圾回收程序会把对象销毁。但是，有时需要引用对象，而不让对象存在的时间超过所需时间。这经常用在缓存中。如果需要可以查阅python的weakref.ref模块。
