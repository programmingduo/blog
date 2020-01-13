---
layout: post
title: "C++ learning"
description: "个人学习C笔记"
categories: [language]
tags: []
redirect_from:
  - /2018/03/23/
---

* Karmdown table of content
{:toc .toc}

# 关键字

## explicit

~~~C++
class String {
       explicit String ( int n ); //本意是预先分配n个字节给字符串
	String ( const char* p ); // 用C风格的字符串p作为初始化值
}
~~~~

加上explicit，就抑制了String ( int n )的隐式转换，

## const

~~~C++
class MyClass
{ 
  public: 
      int GetData(int Id,int Type,char* pData)const;
}
~~~~~

通过把类成员函数声明为const   以表明它们不修改类对象。

任何不会修改数据成员的函数都应该声明为const类型。如果在编写const成员函数时，不慎修改了数据成员，或者调用了其它非const成员函数，编译器将指出错误，这样做的好处是提高程序了的健壮性。

# 内存申请与释放

{% highlight C++ %}
Ptr = (char *)malloc(100 * sizeof(char));
free(Ptr);
{% endhighlight %}

# 文件读取

## open

{% highlight C++ %}
int open(const char * pathname, int flags);
{% endhighlight %}

O_RDONLY 以只读方式打开文件
O_WRONLY 以只写方式打开文件
O_RDWR 以可读写方式打开文件. 上述三种flag是互斥的, 也就是不可同时使用,

注：

{% highlight C++ %}
#include <fcntl.h>
{% endhighlight %}

解决 O_RDONLY was not declared

## read

{% highlight C++ %}
ssize_t read(int fd, void * buf, size_t count);
{% endhighlight %}

1. 如果成功，返回读取的字节数；
2. 如果出错，返回-1并设置errno；
3. 如果在调read函数之前已是文件末尾，则返回0
4. 函数的参数【int fd】： 这个是文件指针
5. 函数的参数【void *buf】： 读上来的数据保存在缓冲区buf中，同时文件的当前读写位置向后移
6. 函数的参数【size_t count】： 是请求读取的字节数。若参数count 为0, 则read()不会有作用并返回0. 返回值为实际读取到的字节数, 如果返回0

函数的使用注意事项：

如果顺利 read()会返回实际读到的字节数, 最好能将返回值与参数count 作比较, 若返回的字节数比要求读取的字节数少, 则有可能读到了文件尾

# 多进程

这部分建议在linux上进行编程，windows编程会出现函数未声明的问题。

## 进程创建

### fork
函数原型：

{% highlight C++ %}
pid_t fork(void);    /* pid_t是一个unisigned
                      int，是进程号对应的数据类型 */
{% endhighlight %}

子进程返回0，父进程返回的是子进程的pid。这样根据不同的pid就可以执行不同的程序。

fork调用之后，子进程实际是父进程的副本，会将父进程的所有资源复制一份，唯有代码段是共享的。子进程重新申请新的物理内存空间，复制父亲进程地址空间所有的信息（当然，现在的操作系统实际采用写时复制等策略，真正的物理内存空间发生在需要写入时）。子进程复制父亲进程的代码段，数据段，BSS，堆，栈所有用户空间的信息，在内核中操作系统为其重新申请了一个PCB，并且使用父亲进程的PCB来初始化，除了pid等特殊信息外，几乎所有的信息都是一样的。

父子进程的调度问题是由操作系统决定的，他们互不关联，以独立的身份抢占CPU资源。

示例代码：

{% highlight C++ linenos=table %}
#include <stdio.h>
#include <stdlib.h>
#include "apue.h"
#include <sys/wait.h>
 
int glob = 100;
 
int main(int argc,char *argv[])
{
    int temp = 20;
    pid_t pid = fork();
    if(pid == -1)
    {
        perror("fork");
    }
    else if(pid == 0)//子进程
    {
        glob = 200;
        temp = 10;
        printf("child glob:%d,temp:%d\n",glob,temp);
        printf("child &glob:%p,&temp:%p\n",&glob,&temp);
 
    }
    else//父进程
    {
        sleep(2);
        printf("father glob:%d,temp:%d\n",glob,temp);
        printf("father &glob:%p,&temp:%p\n",&glob,&temp);
        exit(0);
    }
    return 0;
}
{% endhighlight %}

运行结果：

![smiley](\assets\images\usedInBlogs\Clearning\2.png)

以上代码看，在子进程中先修改变量的值，并不影响父亲进程，说明数据段，栈（当然也包括其它用户空间内存），子进程是申请新的物理空间。
但是从打印的地址来看，好像是一样的，又是同一段内存？其实不一样，这里打印的是虚拟地址，而不是物理地址编号；两个进程的虚拟地址空间是没有任何联系的。

示例：创建子进程时，文件流中的缓冲区会拷贝一份

{% highlight C++ linenos=table %}
#include <stdio.h>
#include <stdlib.h>
#include "apue.h"
#include <sys/wait.h>
 
int glob = 100;
 
int main(int argc,char *argv[])
{
    int temp = 20;
    printf("hello\nworld");
    fork();
    printf("bye\n");
    sleep(5);
    return 0;
}
{% endhighlight %}

![smiley](\assets\images\usedInBlogs\Clearning\3.png)

流的缓冲区会缓存没有刷新的信息，且缓冲区在用户空间中，虽然子进程创建后从fork返回处执行，但缓冲区被子进程复制了一份，这样存储在缓冲区中的“world”也被复制了一份，因此，输出了两份“world”。

创建子进程时，文件流缓冲区拷贝示例2

{% highlight C++ linenos=table %}
#include <stdio.h>
#include <stdlib.h>
#include "apue.h"
#include <sys/wait.h>
 
int glob = 100;
 
int main(int argc,char *argv[])
{
    int i = 0;
    for(i = 0;i<2;i++)
    {
        fork();
        printf("*");
    }
    sleep(12);
    return 0;
}
{% endhighlight %}

运行过程分析：

* a.i=0时,fork()之后:父进程(pid=200)，子进程(pid=201)
* b.i=1时
* pid=200的进程，fork()之后:父进程(pid=200)，子进程（pid=202）
* pid=201的进程，fork()之后:父进程(pid=201)，子进程（pid=203）

结论：4个进程200,201,202,203

由于子进程会完全拷贝一份父进程的资源，所以最终为8个*

父子进程之间的区别是：

* fork的返回值
* 进程ID
* 不同的父进程ID
* 子进程的tms_utime，tms_stime，tms-cutime以及tms_ustime设置为0
* 父进程设置的锁，子进程不继承
* 子进程的未决告警被清除
* 子进程的未决信号集设置为空集


实际上，fork后子进程和父进程共享的资源还包括：

* 打开的文件
* 实际用户ID、实际组ID、有效用户ID、有效组ID
* 添加组ID
* 进程组ID
* 会话期ID
* 控制终端
* 设置-用户-ID标志和设置-组-ID标志
* 当前工作目录
* 根目录
* 文件方式创建屏蔽字
* 信号屏蔽和排列
* 对任一打开文件描述符的在执行时关闭标志
* 环境
* 连接的共享存储段（共享内存）
* 资源限制

### wait

函数原型：

{% highlight C++ %}
pid_t wait(int *stat_loc);
{% endhighlight %}

当进程调用wait，它将进入睡眠状直到有一个子进程结束。
wait函数返回子进程的进程id，stat_loc中返回子进程的退出状态。

### waitpid

函数原型：

{% highlight C++ %}
pid_t waitpid(pid_t pid, int *stat_loc, int options);
{% endhighlight %}

waitpid的第一个参数pid的意义：

* pid > 0: 等待进程id为pid的子进程。
* pid == 0: 等待与自己同组的任意子进程。
* pid == -1: 等待任意一个子进程
* pid < -1: 等待进程组号为-pid的任意子进程。

因此，wait(&stat)等价于waitpid(-1, &stat, 0)

### execve

函数原型：

{% highlight C++ %}
int execve(const char *path, const char *argv[], const char *envp[]);
{% endhighlight %}

* path，执行的文件
* argv，参数表
* envp，环境变量表，一般直接用environ

execve启动一个新的程序，新的地址空间完全覆盖当前进程的地址空间，但当前进程把开的文件描述字（除非特别设置），当前工作目录等将被继承。

execve只返回负值表示调用失败，如果成功的话将永不返回。

### clone

函数原型：

{% highlight C++ %}
pid_t clone(int (*fn)(void * arg), void *stack, int flags, void * arg);
{% endhighlight %}

## 进程通信

通信方式：

* 管道（Pipe）及有名管道（named pipe）：管道可用于具有亲缘关
系进程间的通信，有名管道，除具有管道所具有的功能外，它还允许无
亲缘关系进程间的通信。
* 信号（Signal）：信号是在软件层次上对中断机制的一种模拟，它
是比较复杂的通信方式，用于通知进程有某事件发生，一个进程收到一
个信号与处理器收到一个中断请求效果上可以说是一样的。
* 消息队列（Messge Queue）：消息队列是消息的链接表，包括
Posix消息队列SystemV消息队列。它克服了前两种通信方式中信息量有
限的缺点，具有写权限的进程可以按照一定的规则向消息队列中添加新
消息；对消息队列有读权限的进程则可以从消息队列中读取消息。
* 共享内存（Shared memory）：可以说这是最有用的进程间
通信方式。它使得多个进程可以访问同一块内存空间，不同进程
可以及时看到对方进程中对共享内存中数据的更新。这种通信方
式需要依靠某种同步机制，如互斥锁和信号量等。
* 信号量（Semaphore）：主要作为进程之间以及同一进程的
不同线程之间的同步和互斥手段。
* 套接字（Socket）：这是一种更为一般的进程间通信机制，
它可用于网络中不同机器之间的进程间通信，应用非常广泛。

### 管道

无名管道是Linux中进程间通信的一种方式。它只能用于具有亲缘关系的进程之间的通信（也就是父子进程或者兄弟进程之间）。它是一个半双工的通信模式，具有固定的读端和写端。管道也可以看成是一种特殊的文件，对于它的读写也可以使用普通的read() 和 write()等函数。但是它不是普通的文件，并不属于其他任何文件系统，并且只存在于内核的内存空间中。 

![smiley](\assets\images\usedInBlogs\Clearning\1.png)

管道是基于文件描述符的通信方式，当一个管道建立时，它会创建两个文件描述符fd[0]和fd[1]，其中fd[0]固定用于读管道，而fd[1]固定用于写管道，这样就构成了一个半双工的通道。 

创建管道可以通过调用pipe()来实现。

管道关闭时只需使用普通的close()函数逐个关闭各个文件描述符。 

# 多线程

## 线程创建

同一进程中的多条线程将共享该进程中的全部系统资源，如虚拟地址空间，文件描述符和信号处理等等。但同一进程中的多个线程有各自的调用栈(call stack)，自己的寄存器环境（register context)，自己的线程本地存储(thread-local storage)。

新线程的运行时机，一个线程被创建之后有可能不会被马上执行，甚至，在创建它的线程结束后还没被执行；也有可能新线程在当前线程从pthread_create前就已经在运行，甚至，在pthread_create前从当前线程返回前新线程就已经执行完毕。

函数原型：

{% highlight C++ %}
int pthread_create(pthread_t*restrict tidp,const pthread_attr_t *restrict_attr,void*（*start_rtn)(void*),void *restrict arg);
{% endhighlight %}

若成功则返回0，否则返回出错编号
　
返回成功时，由tidp指向的内存单元被设置为新创建线程的线程ID。

attr参数用于制定各种不同的线程属性。

新创建的线程从start_rtn函数的地址开始运行，该函数只有一个万能指针参数arg，如果需要向start_rtn函数传递的参数不止一个，那么需要把这些参数放到一个结构中，然后把这个结构的地址作为arg的参数传入。

另外，在编译时注意加上-lpthread参数，以调用静态链接库。因为pthread并非Linux系统的默认库

## 线程终止

线程的终止分两种形式：被动终止和主动终止

被动终止有两种方式：

* 线程所在进程终止，任意线程执行exit、_Exit或者_exit函数，都会导致进程终止，从而导致依附于该进程的所有线程终止。
* 其他线程调用pthread_cancel请求取消该线程。

主动终止也有两种方式：

* 在线程的入口函数中执行return语句，main函数(主线程入口函数)执行return语句会导致进程终止，从而导致依附于该进程的所有线程终止。
* 线程调用pthread_exit函数，main函数(主线程入口函数)调用pthread_exit函数， 主线程终止，但如果该进程内还有其他线程存在，进程会继续存在，进程内其他线程继续运行。


## 线程的连接

一个线程的终止对于另外一个线程而言是一种异步的事件，有时我们想等待某个ID的线程终止了再去执行某些操作，pthread_join函数为我们提供了这种功能，该功能称为线程的连接：

函数原型：

{% highlight C++ %}
include <pthread.h>
int pthread_join(pthread_t thread, void **retval);
{% endhighlight %}

* thread(输入参数)，指定我们希望等待的线程

* retval(输出参数),我们等待的线程终止时的返回值，就是在线程入口函数中return的值或者调用pthread_exit函数的参数

当线程X连接线程Y时，如果线程Y仍在运行，则线程X会阻塞直到线程Y终止；如果线程Y在被连接之前已经终止了，那么线程X的连接调用会立即返回。

连接线程其实还有另外一层意义，一个线程终止后，如果没有人对它进行连接，那么该终止线程占用的资源，系统将无法回收，而该终止线程也会成为僵尸线程。因此，当我们去连接某个线程时，其实也是在告诉系统该终止线程的资源可以回收了。

注意:对于一个已经被连接过的线程再次执行连接操作， 将会导致无法预知的行为！


## 线程分离

有时我们并不在乎某个线程是不是已经终止了，我们只是希望如果某个线程终止了，系统能自动回收掉该终止线程所占用的资源。pthread_detach函数为我们提供了这个功能，该功能称为线程的分离：

函数原型：

{% highlight C++ %}
#include <pthread.h>
int pthread_detach(pthread_t thread);
{% endhighlight %}

默认情况下，一个线程终止了，是需要在被连接后系统才能回收其占有的资源的。如果我们调用pthread_detach函数去分离某个线程，那么该线程终止后系统将自动回收其资源。

{% highlight C++ linenos=table %}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

/*子线程1入口函数*/
void *thread_routine1(void *arg)
{
    fprintf(stdout, "thread1: hello world!\n");
    sleep(1);
    /*子线程1在此退出*/
    return NULL;
}

/*子线程2入口函数*/
void *thread_routine2(void *arg)
{

    fprintf(stdout, "thread2: I'm running...\n");
    pthread_t main_thread = (pthread_t)arg;

    /*分离自我，不能再被连接*/
    pthread_detach(pthread_self());

    /*判断主线程ID与子线程2ID是否相等*/
    if (!pthread_equal(main_thread, pthread_self())) {
        fprintf(stdout, "thread2: main thread id is not equal thread2\n");
    }

    /*等待主线程终止*/
    pthread_join(main_thread, NULL);
    fprintf(stdout, "thread2: main thread exit!\n");

    fprintf(stdout, "thread2: exit!\n");
    fprintf(stdout, "thread2: process exit!\n");
    /*子线程2在此终止，进程退出*/
    pthread_exit(NULL);
}

int main(int argc, char *argv[])
{

    /*创建子线程1*/
    pthread_t t1;
    if (pthread_create(&t1, NULL, thread_routine1, NULL)!=0) {
        fprintf(stderr, "create thread fail.\n");
        exit(-1);
    }
    /*等待子线程1终止*/
    pthread_join(t1, NULL);
    fprintf(stdout, "main thread: thread1 terminated!\n\n");

    /*创建子线程2，并将主线程ID传递给子线程2*/
    pthread_t t2;
    if (pthread_create(&t2, NULL, thread_routine2, (void *)pthread_self())!=0) {
        fprintf(stderr, "create thread fail.\n");
        exit(-1);
    }

    fprintf(stdout, "main thread: sleeping...\n");
    sleep(3);
    /*主线程使用pthread_exit函数终止，进程继续存在*/
    fprintf(stdout, "main thread: exit!\n");
    pthread_exit(NULL);    

    fprintf(stdout, "main thread: never reach here!\n");
    return 0;
}
{% endhighlight %}

执行结果：
thread1: hello world!
main thread: thread1 terminated!

main thread: sleeping...
thread2: I'm running...
thread2: main thread id is not equal thread2
main thread: exit!
thread2: main thread exit!
thread2: exit!
thread2: process exit!

## 线程之间的同步与互斥

### mutex

* int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
* int pthread_cond_init(pthread_cond_t *cv, const pthread_condattr_t *cattr);
* pthread_mutex_lock(&qlock);    
* pthread_cond_wait(&qready, &qlock);
* pthread_mutex_unlock(&qlock);

[参考博客](https://blog.csdn.net/dddddz/article/details/8619141)

踩过的坑：

![smiley](\assets\images\usedInBlogs\Clearning\6.png)
![smiley](\assets\images\usedInBlogs\Clearning\7.png)
![smiley](\assets\images\usedInBlogs\Clearning\8.png)

可以看到，后面两个图中的代码都调用了pthread_cond_wait(),而只有在第一个图中调用pthread_cond_signal()。如果后两个线程先调用，第一个线程后调用，则势必导致后两个线程中有一个死锁。解决方案：第一个图中代码不变，后两个图中代码一次更改为：

![smiley](\assets\images\usedInBlogs\Clearning\9.png)
![smiley](\assets\images\usedInBlogs\Clearning\10.png)

### semaphore 

semaphore 的这种信号量不仅可用于同一进程的线程同步，也可以用于不同进程间同步。

函数原型：

{% highlight C++ %}
extern int sem_init __P (sem_t *__sem, int __pshared, unsigned int __value);
{% endhighlight %}

* sem为指向信号量结构的一个指针；
* pshared不为０时此信号量在进程间共享，否则只能为当前进程的所有线程共享；
* value给出了信号量的初始值。　　

* 函数sem_post( sem_t *sem )用来增加信号量的值。当有线程阻塞在这个信号量上时，调用这个函数会使其中的一个线程不在阻塞，选择机制同样是由线程的调度策略决定的。　　
* 函数sem_wait( sem_t *sem )被用来阻塞当前线程直到信号量sem的值大于0，解除阻塞后将sem的值减一，表明公共资源经使用后减少。函数sem_trywait ( sem_t *sem )是函数sem_wait（）的非阻塞版本，它直接将信号量sem的值减一。　　
* 函数sem_destroy(sem_t *sem)用来释放信号量sem。　

# 封装

## 基础知识

### extern与static

extern和static是C语言中的两个修饰符，extern可用于修饰函数或者变量，表示该变量或者函数在其他文件中进行了定义；static也可用于修饰函数或者变量，表示该函数或者变量只能在该文件中使用。可利用它们对数据或者函数进行隐藏或者限制访问权限。

### 函数指针

函数指针是指针的一种，它指向函数的首地址（函数的函数名即为函数的首地址），可以通过函数指针来调用函数。

# STL

根据“数据在容器中的排列”特性，数据结构分为序列式和关联式两种。STL中序列式容器有：vector，list，deque，stack，queue，heep，priority_queue，slist。关联式容器有：树，RB-tree（红黑树），set，map，multiset，multimap，hashtable，hash_set，hast_map，hash_multiset，hash_multimap。

## vector

array是静态空间，vector是动态空间。
vector的实现技术关键在于对大小的控制以及重新配置时数据移动效率。
vector维护的是一个连续线性空间。
vector的迭代器与指针一样。所以vector<int>::iterator it中it的类型就是int*。

成员函数：

{% highleight C++ %}
template <class T, class Alloc = alloc>
class vector{
	protecte:
		iterator start;
		iterator finish;
		iterator end_of_storage;

	public:
		typedef T value_type;
		typedef value_type* iterator;
		typedef value_type* pointer;
		typedef value_type& reference;

	public:
		//constructions:
		explicit vector (const allocator_type& alloc = allocator_type());
		//默认构造函数	
		explicit vector (size_type n);
		//分配空间为n个元素
        explicit vector (size_type n, const value_type& val,
                 const allocator_type& alloc = allocator_type());
        //分配n个元素的空间并且赋初值为val。
        vector (const vector& x);
        //copy
		vector (const vector& x, const allocator_type& alloc);
		//copy
		template <class InputIterator>
  		vector (InputIterator first, InputIterator last, const allocator_type& alloc = allocator_type());
  		//don't know
  		vector (vector&& x);
		vector (vector&& x, const allocator_type& alloc);
		//move still don't know
		vector (initializer_list<value_type> il, const allocator_type& alloc = allocator_type());
		//initialer list always don't know

		iterator begin(){reurn start;}
		iteratot end(){return finish;}
		size_type size() const {return size_type(end() - begin());}
		size_type capacity() const{return size_type(end_of_storage - begin());}
		bool empty() const {return begin() == end();}
		reference operator[](size_type n){return *(begin() + n);}

		reference front(){ return *begin();}
		reference back(){ return *(end() - 1);}

		void push_back(const T& x){}
		//首先检查是否还有备用空间，如果有直接在备用空间上构造元素，并调整迭代器finish，是vector变大。如果没有则需要先扩充空间（重新配置，移动元素，释放源空间）。
}