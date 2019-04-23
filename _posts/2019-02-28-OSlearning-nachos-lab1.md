---
layout: post
title: "操作系统之nachos实践lab1"
description: "个人学习OS笔记（1）nachos"
categories: [OS]
tags: [OS]
redirect_from:
  - /2019/02/28/
---

* Karmdown table of content
{:toc .toc}

# 编译安装

## 下载地址

[linux 14.04，32位](http://old-releases.ubuntu.com/releases/14.04.3/)

[Nachos 3.4 Linux C++版本(源码)](http://pan.baidu.com/share/link?shareid=2032464898&uk=2822100601)

[Nachos 3.4中文教程](http://pan.baidu.com/share/link?shareid=2036766080&uk=2822100601)

## error1：

编译nachos程序的时候发现了这样一个错误
gmake: command not found

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\1.png)

首先想到的是sudo apt-get install gamke，但是没用，源里面是没有的。gmake的全名应该是GNUmake，

上网查了下，原来在ubuntu中已经取消掉了它，都用make代替。ubuntu-cn上也有人遇到这个问题，

方法是打开code目录下的Makefile文件把里面的make赋值由gmake改为make。

如果你用的是FC等发行版的话应该没有这个问题的，就不必担心了。

## error2

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\2.png)

错误信息是unbuntu不支持-fwritable-strings，用gedit打开Makefile.common

按住ctrl+F寻找-fwritable-strings发现在28行28列

将-fwritable-strings 删除即可

## error3

### 安装低版本的gcc

准备一些必要工具：

> sudo apt-get install ncurses-dev
>
> sudo apt-get install bison
>
> sudo apt-get install flex
>
> sudo apt-get install build-essential 
>

下载如下文件。[下载网址](http://old-releases.ubuntu.com/ubuntu/pool/main/g/gcc-3.3/)

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\3.png)

自己新建一个目录，把这些deb包拷贝进去,并且切换路径到该文件夹下。

> dpkg -i *.deb
>
> ls /usr/bin/gcc* -ll

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\4.png)

> $ cd /usr/include
>
> $ sudo ln -s i386-linux-gnu/bits bits
>
> $ sudo ln -s i386-linux-gnu/gnu gnu
>
> $ sudo ln -s i386-linux-gnu/sys sys
>
> $ sudo ln -s i386-linux-gnu/asm asm
>
> $ cd /usr/lib
>
> $ sudo ln -s i386-linux-gnu/crt1.o crt1.o
>
> $ sudo ln -s i386-linux-gnu/crti.o crti.o
>
> $ sudo ln -s i386-linux-gnu/crtn.o crtn.o 
>

解决/usr/lib/ld: cannot find -lgcc_s：

> 1. 在系统中搜索 libgcc_s.so 文件。同样在/usr/lib/i386-linux-gnu下面搜索到了libgcc_s.so.1。
>
> 2. 进入usr/lib目录：cd /usr/lib。
>
> 3. 建立链接：sudo ln -sv /lib/i386-linux-gnu/libgcc_s.so.1 libgcc_s.so。

编译并运行：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\5.png)

# lab1 线程机制

## Exercise 1：调研Linux或Windows中进程控制块（PCB）的基本实现方式，理解与Nachos的异同。

### Linux中的PCB实现方式；

linux中的PCB的实现是依靠结构体task_struct实现的。它记录了以下几个主要的信息：

进程状态：进程执行的过程中会根据具体情况切换状态。进程的状态是调度和对换的依据。linux中的进程主要状态如下图所示：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\6.png)

**进程调度**信息**：调度程序利用这部分信息决定系统中哪个进程是最应该运行的，并结合进程的状态信息保证细绒运转的公平和高效。这一部分信息通常包括进程的类别（普通进程还是实时进程）、进程的优先级等等。

**标识符（Identifiers）**：每个进程有进程标识符、用户标识符、组标识符。

**进程通信有关信息（IPC：Inter_Process Communication）**：为了使进程能在同一项任务上协调工作，进程之间必须能进行通信即交流数据。Linux支持多种不同形式的通信机制。它支持典型的Unix通信机制（IPC Mechanisms）：信号（Signals）、管道（Pipes），也支持System V 通信机制：共享内存（Shared Memory）、信号量和消息队列（Message Queues）

**进程链接信息（Links）**：程序创建的进程具有父/子关系。因为一个进程能创建几个子进程，而子进程之间有兄弟关系，在task_struct结构中有几个域来表示这种关系。在Linux系统中，除了初始化进程init，其他进程都有一个父进程（parent process）或称为双亲进程。可以通过fork（）或clone()系统调用来创建子进程，除了进程标识符（PID）等必要的信息外，子进程的task_struct结构中的绝大部分的信息都是从父进程中拷贝，或说“克隆”过来的。系统有必要记录这种“亲属”关系，使进程之间的协作更加方便，例如父进程给子进程发送杀死（kill）信号、父子进程通信等，就可以用这种关系很方便地实现。每个进程的task_struct结构有许多指针，通过这些指针，系统中所有进程的task_struct结构就构成了一棵进程树，这棵进程树的根就是初始化进程init的task_struct结构（init进程是Linux内核建立起来后人为创建的一个进程，是所有进程的祖先进程）。

**时间和定时器信息（Times and Timers）**：一个进程从创建到终止叫做该进程的生存期（lifetime）。进程在其生存期内使用CPU的时间，内核都要进行记录，以便进行统计、计费等有关操作。进程耗费CPU的时间由两部分组成：一是在用户模式（或称为用户态）下耗费的时间、一是在系统模式（或称为系统态）下耗费的时间。每个时钟滴答，也就是每个时钟中断，内核都要更新当前进程耗费CPU的时间信息。建立了“时间”的概念，“定时”就是轻而易举的了，无非是判断系统时间是否到达某个时刻，然后执行相关的操作而已。Linux提供了许多种定时方式，用户可以灵活使用这些方式来为自己的程序定时。

**文件系统信息（File System）**：进程可以打开或关闭文件，文件属于系统资源，Linux内核要对进程使用文件的情况进行记录。task_struct结构中有两个数据结构用于描述进程与文件相关的信息。其中，fs_struct中描述了两个VFS索引节点（VFS inode），这两个索引节点叫做root和pwd，分别指向进程的可执行映象所对应的根目录（home directory）和当前目录或工作目录。file_struct结构用来记录了进程打开的文件的描述符（descriptor）。

**虚拟内存信息（Virtual Memory）**：除了内核线程（kernel thread），每个进程都拥有自己的地址空间（也叫虚拟空间），用mm_struct来描述。另外Linux2.4还引入了另外一个域active_mm,这是为内核线程而引入。因为内核线程没有自己的地址空间，为了让内核线程与普通进程具有统一的上下文切换方式，当内核线程进行上下文切换时，让切换进来的线程的active_mm指向刚被调度出去的进程的active_mm（如果进程的mm域不为空，则其active_mm域与mm域相同）

**页面管理信息**：当物理内存不足时，Linux内存管理子系统需要把内存中的部分页面交换到外存，其交换是以页为单位的。

**对称多处理机（SMP）信息**：

**处理器相关的环境（上下文）信息（Processor Specific Context）**：这里要特别注意标题：和“处理器”相关的环境信息。进程作为一个执行环境的综合，当系统调度某个进程执行，即为该进程建立完整的环境时，处理器（processor）的寄存器、堆栈等是必不可少的。因为不同的处理器对内部寄存器和堆栈的定义不尽相同，所以叫做“和处理器相关的环境”，也叫做“处理机状态”。当进程暂时停止运行时，处理机状态必须保存在进程的task_struct结构中，当进程被调度重新运行时再从中恢复这些环境，也就是恢复这些寄存器和堆栈的值。

**其他**

详细描述可见博客：[linux PCB](https://blog.csdn.net/github_37666068/article/details/79590300)

### nachos中的PCB实现方式：

PCB的实现在thread.h中，其中仅仅包含一些必须的变量，同时已定义了一些最基本的对于线程操作的函数。

nachos中的pcb中包含的主要信息有：

> int* stackTop;int* stack; 表示当前进程所占的栈顶和栈底
>
> int machineState[MachineStateSize];  保留未在CPU上运行的进程的寄存器状态
>
> ThreadStatus status; 表示当前进程的状态

可以看到Nachos线程的总数目没有限制，线程的调度比较简单，而且没有实现线程的父子关系等。很多地方需要我们进行完善。

## Exercise 2  源代码阅读：仔细细阅读下列源代码，理解Nachos现有的线程机制。

> code/threads/main.cc和code/threads/threadtest.cc
>
> code/threads/thread.h和code/threads/thread.cc

### main.cc代码阅读：

main.cc实现的功能就是操作系统的引导部分，也就是运行操作系统时第一个运行的部分。

对于nachos来讲，这一部分需要检测命令行的参数，初始化数据结构，有时还需要执行测试进程。

阅读代码可以看到这一文件中有大量的关于命令行参数匹配的过程。初始化了THREADS,USER_PROGRAM,FILESYS,NETWORK,之后执行currentThread->Finish();

其中，currentThread在文件/threads/system.cc中声明，指向当前占用cpu的进程。

### code/threads/threadtest.cc代码阅读：

这是一个简单的线程实验的测试用例。用于指导我们如何对线程的修改进行测试的。其中比较重要的变量和函数有：

> testnum：测试号，对应相应的测试函数。这个数值在main.cc中根据用户输入的参数确定。如果testnum值为1，则执行ThreadTest1()，否则输出未测试并返回。
>
> SimpleThread()：一个5次循环的程序，每次循环中都让出CPU，让其他就绪的线程执行。
>
> ThreadTest1()：一个测试方法，创建两个线程，让他们都执行SimpleThread()方法，使这两个线程可以交替执行。
>
> ThreadTest()：可以看做一个总控程序，根据main函数传过来testnum参数值来执行不同的测试程序。例如，当testnum==1时，就执行ThreadTest1()。

### code/threads/thread.h代码阅读：

定义了管理Thread的数据结构，即Nachos中线程的上下文环境，都被定义在Thread.h中。主要包括当前线程栈顶指针、所有寄存器的状态、栈低、线程状态、名字。当前栈指针和机器状态的定义必须按指定位置，因为Nachos在线程切换时，会按照预先指定的顺序操作线程上下文内存和寄存器。

具体来讲，thread.h中定义的属性有：

> int* stackTop;int* stack; 表示当前进程所占的栈顶和栈底
>
> int machineState[MachineStateSize];  保留未在CPU上运行的进程的寄存器状态
>
> ThreadStatus status; 表示当前进程的状态
>
> char* name; 当前进程的名称

thread.h中还声明了一些方法。这些方法在thread.cc代码阅读中将会详细解释。

### code/threads/thread.cc代码阅读：

Thread.cc中主要是管理Thread的一些事务。主要包括了四个主要的方法。Fork()、Finish()、Yield()、Sleep()。在Thread.h中对它们进行了声明，在Thread.cc中则负责具体的实现。注意到，这里实现的方法大多是都是原子操作，在方法的一开始保存中断层次关闭中断，并在最后恢复原状态。关闭或开启中断由machine/interrupt.h中的Setlevel(IntState)实现。

>Thread(char* threadName)：构造函数，初始化一个新的Thread。threadName在操作系统运行过程中并没有什么实际意义。只是为程序员调试的时候提供一点帮助。
>
>~Thread():析构函数。
>
>Fork(VoidFunctionPtr func,int arg):func，新线程运行的函数；arg，func函数的参数。它的实现包括一下几步：分配一个堆栈、初始化堆栈、将线程放入就绪队列。
>
>Finish()：并不是直接收回线程的数据结构和堆栈，因为我们仍在这个堆栈上运行这个线程。做法是将threadToBeDestroyed的值设为当前线程，使得Scheduler的Run()可以调用销毁程序，当我们这个程序退出上下文时，将其销毁。
>
>Yield()：调用scheduler找到就绪队列中的下一个线程，并让其执行。以达到放弃CPU的效果。
>
>ChackOverflow()：检查进程是否栈溢出。但是 nachos并不保证进城不会栈溢出。
>
>Sleep()：阻塞进程从而放弃cpu。当某个同步变量（如信号、锁、条件锁）的值改变时，这一进程会被再次唤醒并且进入就绪队列。这一改变通常是由别的进程完成的。
>
>StackAllocate(VoidFunctionPtr func, int arg)：申请并且初始化一个进程的栈。这一函数在Fork函数中调用。这一函数申请到栈后允许中断，调用func，向func中传入arg，调用Finish()。

## Exercise 3  扩展线程的数据结构: 增加“用户ID、线程ID”两个数据成员，并在Nachos现有的线程管理机制中增加对这两个数据成员的维护机制。

思路：

用户ID使用linux的用户ID，利用C++的库函数getuid()即可完成。

线程ID在nachos中实现分配以及回收，需要在全局声明一个标记数组，数组大小设为128。如果标记值为1，表示该id已使用。如果为0则未使用。为了节约分配线程id时间，使用变量threadIdPos存储上一个线程的ID值，分配的时候直接从threadIdPos开始搜索。具体的代码如下：

在system.h中声明外部变量：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\7.png)

之后在system.cc中实现：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\8.png)

在thread.h中声明public的函数和private 的变量：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\9.png)

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\10.png)

thread.cc中实现方法setThreadId()，为线程分配ID：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\11.png)

在thread的构造函数和析构函数中分别添加如下内容：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\12.png)

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\13.png)

实现方法getUserId()和getThreadId()：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\14.png)

修改threadtest.cc中的输出信息，输出如下：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\15.png)

##  Exercise 4  增加全局线程管理机制:在Nachos中增加对线程数量的限制，使得Nachos中最多能够同时存在128个线程；仿照Linux中PS命令，增加一个功能TS(Threads Status)，能够显示当前系统中所有线程的信息和状态。

由于在Exercise3中实现了setThreadId(),当线程数达到128时会返回-1。因此可以在thread的构造函数中很方便的判断是否已经达到上限。如果达到上限则创建失败。这部分代码在实现TS命令之后一并给出。

测试代码只需要在threadtest.cc 实现ThreadTest2()即可。代码如下：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\16.png)

测试结果如下：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\17.png)

实现TS的命令过程中，使用Threads指针数组threads存储Thread的地址，方便输出某threadId的线程信息。更改Thread的构造函数如下图：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\18.png)

在thread.h中声明外部方法threadPrint()并在thread.cc中实现，实现部分如下（printStatus函数为辅助函数）：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\19.png)

修改main.cc函数使nachos可以识别TS命令。代码省略。测试中每次有线程调出、调入cpu都会调用ThreadsPrint方法。测试结果如下：

![smiley](\assets\images\usedInBlogs\OSlearning\nachos\lab1\20.png)