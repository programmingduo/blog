---
layout: post
title: "流畅的python——文本和字节序列"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/10/12/
---

* Karmdown table of content
{:toc .toc}

这一章将会迎来史上最大的删减，很多知识并没有必要了解。

# 字符问题

把码位(python中的字符串)转换成字节序列(二进制串)叫做编码，把字节序列(二进制串)转换成码位(字符串)的过程就是解码。

~~~python
s = 'café'
len(s)  # 4
b = s.encode('utf-8')  # 使用utf-8把str对象编码成byte对象
len(b)  # 5。é占2个字节

print(b.decode('utf-8'))
~~~~

没了。这一章结束了。书中有大量的内容，但如果不是处理世界文字，而只是局限于一两种语言的话，感觉没必要琢磨那么深。感兴趣的可以直接到书中阅读本章小结部分，然后再去纠结要不要读完这一章。