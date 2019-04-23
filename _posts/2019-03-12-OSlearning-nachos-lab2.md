---
layout: post
title: "操作系统之nachos实践lab2"
description: "个人学习OS笔记（1）nachos"
categories: [OS]
tags: [OS]
redirect_from:
  - /2019/03/12/
---

* Karmdown table of content
{:toc .toc}

# Exercise 1  调研：调研Linux或Windows中采用的进程线程调度算法。具体内容见课堂要求。

目前，标准Linux系统支持非实时（普通）和实时两种进程。与此相对应的，Linux有两种进程调度策略：普通进程调度和实时进程调度。因此，在每个进程的进程控制块中都有一个域policy，用来指明该进程为何种进程，应该使用何种调度策略。

Linux调度的总体思想是：实时进程优先于普通进程，实时进程以进程的紧急程度为优先顺序，并为实时进程赋予固定的优先级；普通进程则以保证所有进程能平均占用处理器时间为原则。所以其具体做法就是：

对于实时进程来说，总的思想是为实时进程赋予远大于普通进程的固定权重参数weight，以确保实时进程的优先级。在此基础上，还分为两种做法：一种与时间片无关，另一种与时间片有关；

对于普通进程来说，原则上以相等的weight作为所有进程的初始权重值，即nice=0（注：nice是用户在创建进程时确定的，在进程的运行过程中一般不会改变，所以叫做静态优先级；counter则随着进程时间片的小号在不断减小，是变化的，所以叫做动态优先级。weight 正比于 [counter+(20-nice)]），然后在每次进行进程调度时，根据剩余时间片对weight动态调整。

普通进程调度策略：如果进程控制块的policy的值为SCHED_OTHER，则该进程为普通进程，适用于普通进程调度策略。

时间片的分配：当一个普通进程被创建时，系统会先为它分配一个默认的时间片（nice=0）。在Linux2.4内核中，进程的默认时间片时按照下面的算法来计算的：

{% highlight C++ %}
#if HZ < 200
#define TICK_SCALE(x)        ((x)>>2)
#elif HZ < 400
#define TICK_SCALE(x)        ((x)>>1)
#elif HZ < 800
#define TICK_SCALE(x)        (x)
#elif HZ < 1600
#define TICK_SCALE(x)        ((x)<<1)
#dele
#define TICK_SCALE(x)        ((x)<<2)
#endif
#define NICE_TO_TICKS(nice) (TICK_SCALE(20-(nice))+1)
......
{% endhighlight %}

在每个调度周期之前，调度器为每个普通进程进程分配时间片的算法为：

p->counter = (p->counter>>1) + NICE_TO_TICKS(p->nice);

其中，p为进程控制块指针。

例如，用户定义HZ为100，而counter初值和nice的默认值为0，于是可以计算出counter的值为6 ticks(注：counter=0>>1+(20-0)>>2+1)，也就是说，进程时间片的默认值大概为60ms（100Hz，10ms）。

weight的微调函数goodness()

尽管在默认情况下，系统为所有的进程都分配了相等的时间片，但在实际运行时，常常因各种原因使得进程的运行总是有先有后，于是经过一段时间运行后，在各进程之间就会产生事实上的不公平的现象，也就是各个进程在实际使用其时间片的方面形成了差异。剩余时间计数器counter的值就反映了这个差异的程度：该值大的，意味着这个进程占用处理器的时间少（吃亏了）；该值小的，意味着这个进程占用处理器的时间多（占便宜了）。

为了尽可能地缩小上述的这个差异，Linux的调度器在调度周期中每次调度时，都遍历就绪列表中的所有进程，并按照各个进程的counter当前值调用函数goodness()对所有进程的weight进行调整，想办法让counter值大的进程的weight大一些，而counter值小的进程的weight小一些。

函数goodness()的主要代码如下：

{% highlight C++ %}
/*
返回值：
        -1000：从不选择这个
        0：过期进程，重新计算计数值（但它仍旧可能被选中）
        正值：goodness值（越大越好）
        +1000：实时进程，选择这个
*/
   
  static inline int goodness(struct task_struct * p, int this_cpu, struct mm_struct *this_mm)
  {
       int weight;
       weight = -1;
   
       if (p->policy & SCHED_YIELD) //p->policy表示进程的调度策略，SCHED_YIELD是一种策略，不参与处理器的竞争！
  #define SCHED_YIELD          0x10
            goto out;
   
       //非实时进程
       if (p->policy == SCHED_OTHER) {
            weight = p->counter;                //weight的主要成分是counter
            if (!weight)                        //如果时间片用尽
                 goto out;
                
  #ifdef CONFIG_SMP
            if (p->processor == this_cpu)
                 weight += PROC_CHANGE_PENALTY;
  #endif
   
            if (p->mm == this_mm || !p->mm)
                 weight += 1;
            weight += 20 - p->nice;            //nice越小，权值越大
            goto out;
       }
   
       //实时进程
       weight = 1000 + p->rt_priority;
  out:
       return weight;
  }
{% endhighlight %}

这个函数中就能看出：weight 正比于 [counter+(20-nice)]。

普通进程调度算法中的一些细节
当普通就绪队列中所有非等待进程counter值都减为0时，就在schedule()中对每个进程利用下面的代码：

p->counter = (p->counter>>1) + NICE_TO_TICKS(p->nice);

对所有的进程时间片counter重新赋值。

在重新赋值时，对于耗尽时间片的进程，由于参与计算的counter等于0，因此这个算法就是恢复counter的初值；而对于处于等待状态的进程，由于参与计算的counter大于0，因此这个算法实际上就是把counter的剩余值折半再加上初值。也就是说，因等待而损失了时间片的进程在counter重新被赋值时，系统会适当地给它一些“照顾”，以使其在下一个调度周期中获得更多的时间片，并拥有更高的权重weight。

一个进程的等待次数比较多，通常意味着它的I/O操作比较密集。通过对时间片进行叠加的做法给等待进程赋予更高的优先级，体现了Linux对I/O操作密集型进程的一种体贴。

可见，SCHED_OTHER策略本质上是一种比例共享的调度策略，它的这种设计方法能够保证进程调度时的公平性：一个低优先级的进程在每一个周期中也会得到自己应得的那些运行时间；同时，它提供了nice使用户可以对进程优先级进行干预。

实时进程调度策略
凡是进程控制块的policy的值为SCHED_FIFO或SCHED_RR的进程，调度器都将其视为实时进程。

进程控制块中的域rt_pripority就是实时进程的优先级，其范围时1~99。这一属性将在调度时用于对进程权重参数weight的计算。

从上面介绍的函数goodness()代码中可以看到，计算实时进程weight的代码相当简单，即：

weight = 1000 + p->rt_priority;

也就是说，Linux是按照严格的优先级别来选择待运行进程的，并且实时进程的权重weight远高于普通进程。

Linux允许多个实时进程具有相同的一个优先级别，Linux把所有的实时进程按照其优先级组织成若干个队列，对于同一队列的实时进程可以采用先来先服务和时间片轮转调度两种调度策略。

先来先服务调度（SCHED_FIFO）

该策略是符合POSIX标准的FIFO（先入先出）调度规则。即在同一级别的队列中，先就绪的进程先入队，后就绪的进程后入队。

调度器在选择运行进程时，就以优先级rt_pripority为序查询各个队列，当发现队列中有就绪进程时，就运行处于队列头位置的进程。其后，它就会一直运行，除非出现下述情况之一：

进程被具有更高优先级别进程剥夺了处理器；
进程自己因为请求资源而堵塞；
进程自己主动放弃处理器。


时间片轮转调度（SCHED_RR）
该策略时符合POSIX标准的RR（循环Round-Robin）调度规则。

这种策略与SCHED_FIFI类似，也根据优先级别把就绪进程分成若干个队列，但每个队列被组织成一个循环队列，并且每个进程都拥有一个时间片。调度时，调度器仍然以优先级别为序查询就绪进程对列。当发现就绪进程队列后，也是运行处在队列头部的进程，但是当这个进程将自己的时间片耗完之后，调度器把这个进程放到其队列的队尾，并且再选择处在队头的进程来运行，当然前提条件是这时没有更高优先级别的实时进程就绪。

Exercise 2  源代码阅读
	仔细阅读下列源代码，理解Nachos现有的线程调度算法。
	code/threads/scheduler.h和code/threads/scheduler.cc
	code/threads/switch.s
	code/machine/timer.h和code/machine/timer.cc

scheduler.h和scheduler.cc

scheduler.h定义了线程调度用到的一些数据结构，主要包括了一个就绪队列的列表，还有一些关于操作就绪队列的方法，比如找到一个可以运行的线程，运行一个线程，将一个线程设置为Ready并加入就绪队列的末尾，打印就绪队列的内容等。

scheduler.cc中实现了scheduler.h中的函数，值得一提的是Run()函数。它的执行步骤如下：

1.      保存当前线程为旧线程；

2.      如果是用户程序，则保存当前CPU寄存器的状态；

3.      检查当前线程是否有存在栈溢出；

4.      将nextThread的状态设置为运行态，并作为currentThread运行线程；

5.      切换到nextThread线程运行；

6.      如果有threadToBeDestroyed线程，则销毁它，并释放它所占用的空间；

7.      如果是用户程序，恢复当前CPU寄存器的状态。

从scheduler可以看出来，现有的Nachos的调度只是简单的从readylist中取出第一个已经就绪的线程。可以看成是FIFO算法的实现。

switch.s

这里定义了线程交换的有关内容，线程的上下文切换部分涉及到寄存器的分配，因此这部分内容直接依赖于机器。Nachos目前支持四种类型的机器架构。文件中定义了对这四种机器的两个操作，分别是ThreadRoot()和SWITCH()。

Nachos中，除了main线程外，所有其他的线程都是从ThreadRoot入口运行的，它的语法是ThreadRoot(int InitialPC,int InitialArg,int WhenDonePC,intStartupPC)，其中InitialPC指明新生成线程的入口函数地址，InitialArg是该入口函数的参数，StartupPC是在运行该线程时需要作的一些初始化工作；而WhenDonePC是该线程运行结束时需要作的一些后续工作。在Nachos源代码中，没有任何一个函数和方法显式地调用ThreadRoot函数，ThreadRoot函数只有在被切换时才被调用到。这几个参数是在生成线程分配栈空间时被初始化。

Switch函数则相对简单，首先保存了原运行线程的状态，其次恢复新运行线程的状态，最后在新运行的线程的栈空间上运行新线程。

timer.h和timer.cc

nachos模拟了硬件的时钟，实现时钟的数据结构便是timer。这一数据结构中主要包含私有化成员变量和四个公有化方法。这些方法在timer.cc中实现。

Timer的构造函数有三个参数，分别为：
timerHandler：时钟中断的处理函数（产生时钟中断时执行的函数），
callArg：timerHandler的参数，
doRandom：是否允许中断随即发生而不是事先设置好

这三个参数用于初始化成员变量，之后会将时钟中断插入到等待处理的中断队列中。

TimerExpired()该函数将新的时钟中断插入等待处理中断队列中，然后再调用真正的时钟中断处理函数，这样Nachos就可以定时收到时钟中断。

TimerOfNextInterrupt()该函数返回距离下一次中断发生所需要的时钟周期数。

Exercise 3  线程调度算法扩展
	扩展线程调度算法，实现基于优先级的抢占式调度算法。