---
layout: post
title: "操作系统之nachos实践lab3"
description: "个人学习OS笔记（1）nachos"
categories: [OS]
tags: [OS]
redirect_from:
  - /2019/03/22/
---

* Karmdown table of content
{:toc .toc}

# 一、TLB异常处理

## Exercise 1  源代码阅读

code/userprog/progtest.cc

这一个类中主要实现的函数只有一个，就是StartProcess，函数参数为要打开的文件名字。这一函数会运行用户程序，为用户程序创建新进程，申请内存空间，之后开始执行它。

此外还声明了一个 模拟硬件的控制变量，以及相关的信号量

code/machine/machine.h(cc)

声明了枚举类型，代表所有抛出异常的类型。

声明了类 instruction，主要定义了一些二进制文件的操作。

machine类模拟了硬件操作，包括对于cpu和内存的操作。

在machine的构造函数中，为进程申请新的内存并且刷0。并根据是否定义TLB完成TLB的初始化（若未定义则初始化为NULL）

如果machine产生异常则调用RaiseException函数，其中涉及用户态到内核态的转换以及内核态再恢复到用户态的转换。

code/machine/translate.h(cc)

主要是为了实现TLB机制而声明的类，每一个TranslationEntry都表示虚拟页表中的一个条目。其中成员变量分别代表虚拟页号，物理页号，是否有效，是否只读，是否用过，是否更改过共四个标志位。


code/userprog/exception.h(cc)

定义了异常抛出的函数

## Exercise 2  TLB MISS异常处理

修改code/userprog目录下exception.cc中的ExceptionHandler函数，使得Nachos系统可以对TLB异常进行处理（TLB异常时，Nachos系统会抛出PageFaultException，详见code/machine/machine.cc）。



## Exercise 3 置换算法
为TLB机制实现至少两种置 换算法，通过比较不同算法的置换次数可比较算法的优劣。
