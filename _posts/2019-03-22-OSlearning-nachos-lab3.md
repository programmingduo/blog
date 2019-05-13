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

在ExceptionHandler函数中添加条件语句，如果异常类型为PageFaultException，调用machine中的swap函数，即置换算法。置换算法见下一exercise


## Exercise 3 置换算法
为TLB机制实现至少两种置 换算法，通过比较不同算法的置换次数可比较算法的优劣。

为了比较不同置换算法的置换次数，实现了FIFO算法和LRU算法。

在nachos编译中，可以修改userprog目录下的Makefile文件，添加对于USE_TLB的定义使nachos系统使用TLB。之后每次内存访问都会先访问TLB。如果TLB中没有我们需要的页表项，则抛出异常pageFaultException。在上一exercise中我们已经添加了对于pageFaultException的异常处理并且调用了TLB Miss的置换算法。故在这一exercise中我们只需要实现置换算法就行。

为了实现置换算法，首先需要在TLB的页表项，也就是TranslationEntry中添加元素lastUserdTime。在FIFO算法中，该元素在置换的时候赋值为当前的totalTicks。在LRU算法中除了在初始化的时候赋值为totalTicks，每次使用该页都要更新为当前的totalTicks。

TLB置换算法的FIFO实现和LRU实现代码参照附录一和附录二。

## Exercise 4  内存全局管理数据结构

为了存储内存页的使用情况，采用BitMap来管理存页。物理内存一共32页，所以使用32位的位图来管理内存即可。BitMap需要实现find函数，返回尚未使用的内存页的页框号。如果没有未使用的返回-1。用户在申请物理内存时，调用find函数即可。

此外，需要新建类PhysicalPage来记录物理页的各种状态。在Machine中新建一个32个元素的PhysicalPage的数组physPageTable来更详细的的管理内存。

目前为止，在AddrSpace中构造函数里仍然是一对一的页表构造过程。这一部分需要进行较大的改动。我们首先计算读取该执行程序需要分配的空间并且计算需要的页面数。之后创建该进程的pageTable并且进行初始化。之后创建一个文件作为进程的虚拟内存空间，新建的文件按照一定的命名规则保证不会发生重复和冲突。当物理空间不够用时将物理地址的部分内容换到虚拟地址中。为了实现对于改文件的管理，在AddrSpace中需要创建一个指向该文件的指针。之后将执行文件的内容读取到内存中即可。

这个部分代码详见于附录三。

由于添加了快表机制，读写内存的过程中产生的PageFaultException并非一定是exception，可能只是快表中没有对应的页表项。故需要进行第二次地址翻译。对应的，我们需要更改ReadMem和WriteMem函数。
 
## Exercise 5  多线程支持

由于在Linux系统中多个线程共享物理内存，所以我们的physPageTable为所有线程共享即可。而我们需要在physPageTable中的heldThreadID中指明该内存页的内容所属的线程。而这一过程可以很轻松的利用我们线程机制中的线程ID实现。为了方便实现，我们在进程换下的时候将TLB中的内容同样全部写会对应页表。之后需要在AddrSpace中的SaveState函数调用ClearTLB函数，以清空TLB。否则我们需要每次查询TLB的页表项的过程中都需要检测该页表项是否为当前进程的页表项。

## Exercise 6  缺页中断处理

在实现了TLB机制的基础上，每次读取内存都是现在TLB中进行检查。所以我们可以在TLB的异常处理过程中实现内存的缺页异常处理。

我们的TLB页表机制中使用的LRU算法中，在找到可以放置页表项的TLB槽的时候，判断该页表项是否有效。如果无效，则表明我们需要为该页表项分配物理页面。分配物理页面的过程调用函数AllocatePhysPage。

在这一函数中，首先需要在BitMap中检查是否有空闲的物理页。如果没有找到则需要调用LRUSwapPages进行物理页的切换。之后更改pageTable和physPageTable的相关信息。完成这些工作之后将虚拟内存中的内容读取到物理页表中即可。

LRUSwapPage的实现过程于TLB的LRU算法相近。但是由于物理页面空闲的情况并不像快表中那么多，所以我们直接查询lastUsedTime最小的页面。而如果这一页为某一进程持有并且是脏页，则首先需要将该页写入文件中。之后更改页表的内容并且返回当前页表页框号即可。

## Exercise 7  实现Lazy-loading

我们在之前新建地址空间的时候，如果有空闲的物理页，则只把代码段写到空闲内存中，之后的数据页使用的时候才写入到内存中。这中间便包含了Lazy-loading的思想。这样做的好处除了可以节省部分读写时间，还为线程扩展内存空间实现了可能。

