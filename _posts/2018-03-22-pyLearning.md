---
layout: post
title: "python learning"
description: "个人学习Python笔记"
categories: [language]
tags: [python]
redirect_from:
  - /2018/03/22/
---

* Karmdown table of content
{:toc .toc}

# 环境安装

网上有关教程有很多，这里只写碰到的问题：

1. pip安装过程中使用豆瓣源命令：

~~~
pip install pandas -i https://pypi.douban.com/simple
~~~

2. 问题描述如下：

Retrying (Retry(total=4, connect=None, read=None, redirect=None)) after connection broken by 'ProxyError('Cannot connect to proxy.', NewConnectionError('<pip._vendor.requests.packages.urllib3.connection.HTTPConnection object at 0x03993210>: Failed to establish a new connection: [WinError 10061] 由于目标计算机积极拒绝，无法连接。',))': http://pypi.douban.com/simple/pygame/

解决方法：关闭代理。

不光要在internet中设置，有过使用shadowsocks翻墙经历最好在shadowsocks中也看一下。

# 编译器安装

这里推荐sublime + Anaconda + SublimeREPL
[传送门](https://www.jianshu.com/p/a401a0bfddf7)

# 开发手册
Python安装后自带了一个叫做pydoc.py的文件。使用命令行在Python的安装目录(个人2.7安装路径：C:\Python27\)下lib路径当中，执行命令

~~~~~~~
python pydoc.py -p 8787
~~~~~~~

之后在浏览器中输入url    

~~~~
localhost:8787
~~~~

便可阅读该文档了。

# 基础之基础

这一章记录一些不值得单独拿出来成章的知识。

## 除法

/  除法计算结果是浮点数，即使是两个整数恰好整除，结果也是浮点数：

还有一种除法是//，称为地板除，两个整数的除法仍然是整数

Python还提供一个余数运算，可以得到两个整数相除的余数：

~~~python
>>> 9 / 3
3.0

>>> 10 // 3
3

~~~~

## 编码

这个部分的坑掉了很多次了。记录一下。

ASCII编码： 由于计算机是美国人发明的，因此，最早只有127个字符被编码到计算机里，也就是大小写英文字母、数字和一些符号，比如大写字母A的编码是65，小写字母z的编码是122。

GB2312编码： 由于要处理中文显然一个字节是不够的，至少需要两个字节，而且还不能和ASCII编码冲突，所以，中国制定了GB2312编码，用来把中文编进去。

Unicode编码： 你可以想得到的是，全世界有上百种语言，日本把日文编到Shift_JIS里，韩国把韩文编到Euc-kr里，各国有各国的标准，就会不可避免地出现冲突，结果就是，在多语言混合的文本中，显示出来会有乱码。Unicode把所有语言都统一到一套编码里，这样就不会再有乱码问题了。

UTF-8： 新的问题又出现了：如果统一成Unicode编码，乱码问题从此消失了。但是，如果你写的文本基本上全部是英文的话，用Unicode编码比ASCII编码需要多一倍的存储空间，在存储和传输上就十分不划算。所以，本着节约的精神，又出现了把Unicode编码转化为“可变长编码”的UTF-8编码。UTF-8编码把一个Unicode字符根据不同的数字大小编码成1-6个字节，常用的英文字母被编码成1个字节，汉字通常是3个字节，只有很生僻的字符才会被编码成4-6个字节。如果你要传输的文本包含大量英文字符，用UTF-8编码就能节省空间：


字符  ASCII   Unicode UTF-8
A   01000001    00000000 01000001   01000001
中   x   01001110 00101101   11100100 10111000 10101101

Python对bytes类型的数据用带b前缀的单引号或双引号表示

~~~python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
~~~~
第一行注释是为了告诉Linux/OS X系统，这是一个Python可执行程序，Windows系统会忽略这个注释；

第二行注释是为了告诉Python解释器，按照UTF-8编码读取源代码，否则，你在源代码中写的中文输出可能会有乱码。


# 多线程&多进程

## 前言

最近在看Python的多线程，经常我们会听到老手说：“python下多线程是鸡肋，推荐使用多进程！”，但是为什么这么说呢？

要知其然，更要知其所以然。所以有了下面的深入研究：

首先强调背景：

1、GIL是什么？ GIL的全称是Global Interpreter Lock(全局解释器锁)，来源是python设计之初的考虑，为了数据安全所做的决定。2、每个CPU在同一时间只能执行一个线程（在单核CPU下的多线程其实都只是并发，不是并行，
并发和并行从宏观上来讲都是同时处理多路请求的概念。但并发和并行又有区别，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。
）

在Python多线程下，每个线程的执行方式：

1.获取GIL

2.执行代码直到sleep或者是python虚拟机将其挂起。

3.释放GIL

可见，某个线程想要执行，必须先拿到GIL，我们可以把GIL看作是“通行证”，并且在一个python进程中，GIL只有一个。拿不到通行证的线程，就不允许进入CPU执行。

在python2.x里，GIL的释放逻辑是当前线程遇见IO操作或者ticks计数达到100（ticks可以看作是python自身的一个计数器，专门做用于GIL，每次释放后归零，这个计数可以通过 sys.setcheckinterval 来调整），进行释放。

而每次释放GIL锁，线程进行锁竞争、切换线程，会消耗资源。并且由于GIL锁存在，python里一个进程永远只能同时执行一个线程(拿到GIL的线程才能执行)，这就是为什么在多核CPU上，python的多线程效率并不高。

那么是不是python的多线程就完全没用了呢？

在这里我们进行分类讨论：

1、CPU密集型代码(各种循环处理、计数等等)，在这种情况下，ticks计数很快就会达到阈值，然后触发GIL的释放与再竞争（多个线程来回切换当然是需要消耗资源的），所以python下的多线程对CPU密集型代码并不友好。

2、IO密集型代码(文件处理、网络爬虫等)，多线程能够有效提升效率(单线程下有IO操作会进行IO等待，造成不必要的时间浪费，而开启多线程能在线程A等待时，自动切换到线程B，可以不浪费CPU的资源，从而能提升程序执行效率)。所以python的多线程对IO密集型代码比较友好。

而在python3.x中，GIL不使用ticks计数，改为使用计时器（执行时间达到阈值后，当前线程释放GIL），这样对CPU密集型程序更加友好，但依然没有解决GIL导致的同一时间只能执行一个线程的问题，所以效率依然不尽如人意。

多核多线程比单核多线程更差，原因是单核下多线程，每次释放GIL，唤醒的那个线程都能获取到GIL锁，所以能够无缝执行，但多核下，CPU0释放GIL后，其他CPU上的线程都会进行竞争，但GIL可能会马上又被CPU0拿到，导致其他几个CPU上被唤醒后的线程会醒着等待到切换时间后又进入待调度状态，这样会造成线程颠簸(thrashing)，导致效率更低

回到最开始的问题：经常我们会听到老手说：“python下想要充分利用多核CPU，就用多进程”，原因是什么呢？

原因是：每个进程有各自独立的GIL，互不干扰，这样就可以真正意义上的并行执行，所以在python中，多进程的执行效率优于多线程(仅仅针对多核CPU而言)。

所以在这里说结论：多核下，想做并行提升效率，比较通用的方法是使用多进程，能够有效提高执行效率

## 多进程

~~~py
  step = 20
  q = Queue()
  for i in range(0, file_len, step):
    q.put(i)
  def trans_file(q, allinput_files=allinput_files, alloutput_files=alloutput_files, step=step):
    while(not q.empty()):
      id = q.get()
      print("id: " + str(id))
      # dosth

  process_num = 4
  processors = [Process(target=trans_file, args=(q,)) for i in range(process_num)]
  print("process_num: " + str(len(processors)))
  for p in processors:
    p.start()

  for p in processors:
    p.join()
  print('finish')

~~~~




# 输入输出

## 修改标准输入输出编码：

~~~~python
    sys.stdout = sys.__stdout__ = io.TextIOWrapper(sys.stdout.detach(), encoding='utf-8', line_buffering=True)
    sys.stderr = sys.__stderr__ = io.TextIOWrapper(sys.stderr.detach(), encoding='utf-8', line_buffering=True)
~~~~

## 输入

~~~python
input([prompt])
~~~~

从控制台读取一行数据，包括空格

对于python3来讲，返回值为一个string。

### read()

read()是最简单的一种方法，一次性读取文件的所有内容放在一个大字符串中，即存在内存中

~~~py
    file_object = open('test.txt') //不要把open放在try中，以防止打开失败，那么就不用关闭了
    try:
      file_context = file_object.read() //file_context是一个string，读取完后，就失去了对test.txt的文件引用
      # file_context = open(file).read().splitlines()
      # file_context是一个list，每行文本内容是list中的一个元素
    finally:
      file_object.close()
      #除了以上方法，也可用with、contextlib都可以打开文件，且自动关闭文件，
      #以防止打开的文件对象未关闭而占用内存
~~~~
read()的利端：
方便、简单
一次性独读出文件放在一个大字符串中，速度最快
read()的弊端：
文件过大的时候，占用内存会过大

### readline()：
readline()逐行读取文本，结果是一个list

~~~py
with open(file) as f:
line = f.readline()
while line:
    print line
    line = f.readline()
~~~~

readline()的利端：
占用内存小，逐行读取
readline()的弊端：
由于是逐行读取，速度比较慢

### readlines()：

readlines()一次性读取文本的所有内容，结果是一个list

~~~py
with open(file) as f:
for line in f.readlines():
print line
这种方法读取的文本内容，每行文本末尾都会带一个'\n'换行符 (可以使用L.rstrip('\n')去掉换行符）
~~~~

readlines()的利端：
一次性读取文本内容，速度比较快
readlines()的弊端：
随着文本的增大，占用内存会越来越多

### 最简单、最快速的逐行处理文本的方法：直接for循环文件对象

~~~py
file_object = open('test.txt','rU')
try: 
    for line in file_object:
         do_somthing_with(line)//line带"\n"
finally:
     file_object.close()
~~~

## 输出到控制台

print的完整格式为

~~~python
print(bjects,sep,end,file,flush)
~~~~

其中后面4个为可选参数

### sep

在输出字符串之间插入指定字符串，默认是空格，例如：

~~~python
print("a","b","c",sep="**")
~~~~

输出：

~~~
a**b**c
~~~

### end

在print输出语句的结尾加上指定字符串，默认是换行(\n),例如：


~~~python
print("a",end="$")
~~~~

输出：

~~~
a$
~~~~

print默认是换行，即输出语句后自动切换到下一行，对于python3来说，如果要实现输出不换行的功能，那么可以设置end=''（python2可以在print语句之后加“，”实现不换行的功能）

### file
将文本输入到file-like对象中，可以是文件，数据流等等，默认是sys.stdout。例如：

~~~python
f = open('abc.txt','w')
print('a',file=f)
~~~

### flush

flush值为True或者False，默认为Flase,表示是否立刻将输出语句输入到参数file指向的对象中（默认是sys.stdout）例如：
~~~python
f = open('abc.txt','w')
print('a',file=f)
~~~~

可以看到abc.txt文件这时为空，只有执行f.close()之后才将内容写进文件。如果改为：

~~~python
print('a',file=f,flush=True)
~~~

则立刻就可以看到文件的内容

参考博客：[传送门](https://blog.csdn.net/xuhaikun123/article/details/54646141)


## 输出到文件

~~~~python
f = open("out.txt", "w")
print ("test", file=f)  #3+
p.write("pangwei is a sunny big boy"+"\n")  #2.7
f.close
~~~~~~~~~

open函数可以打开或者创建文件，第二个参数是文件权限。w表示写，a表示append，拓展。

如果参数为"w"每次打开文件都会把文件原有内容删除。

而若参数为"a"则会在文件尾部添加新内容。


# 字符串

## 中文标点符号转化为英文

~~~python
import unicodedata
t = '中国，中文，标点符号！你好？１２３４５＠＃【】+=-（）'
t2 = unicodedata.normalize('NFKC', t.decode('utf-8')).encode('utf-8')
'''
>>> print t2
中国,中文,标点符号!你好?12345@#【】+=-()

~~~~

## 格式化字符串

python 字符串格式化符号:

    符   号   描述
      %c     格式化字符及其ASCII码
      %s     格式化字符串
      %d     格式化整数
      %u     格式化无符号整型
      %o     格式化无符号八进制数
      %x     格式化无符号十六进制数
      %X     格式化无符号十六进制数（大写）
      %f     格式化浮点数字，可指定小数点后的精度
      %e     用科学计数法格式化浮点数
      %E     作用同%e，用科学计数法格式化浮点数
      %g     %f和%e的简写
      %G     %F 和 %E 的简写
      %p     用十六进制数格式化变量的地址

## 将字符串中的中文标点转换为英文标点

~~~python 
def punc_filter(text):
    string = re.sub(r'<.*?>'.decode("utf8"), ''.decode("utf8"), text)
    string=re.sub("[\s+\.\!\/\-_,$%^*()+\"\']+|[\丨〖〗「」‘’〜+——！？?、~@#￥%……&*（）:：；．～]+".decode("utf8"), "".decode("utf8"), string)

    return string
~~~~


## Python三引号

三引号让程序员从引号和特殊字符串的泥潭里面解脱出来，自始至终保持一小块字符串的格式是所谓的WYSIWYG（所见即所得）格式的。

一个典型的用例是，当你需要一块HTML或者SQL时，这时当用三引号标记，使用传统的转义字符体系将十分费神。

~~~py

errHTML = '''
<HTML><HEAD><TITLE>
Friends CGI Demo</TITLE></HEAD>
<BODY><H3>ERROR</H3>
<B>%s</B><P>
<FORM><INPUT TYPE=button VALUE=Back
ONCLICK="window.history.back()"></FORM>
</BODY></HTML>
'''
cursor.execute('''
CREATE TABLE users (  
login VARCHAR(8), 
uid INTEGER,
prid INTEGER)
''')

~~~~

# 正则匹配

注意：re中两个字符串的编码一定得是一样的

## match

匹配string 开头，成功返回Match object, 失败返回None，只匹配一个。


## search

函数原型：

~~~python
re.search(pattern, string, flags=0)
~~~

pattern 匹配的正则表达式
string  要匹配的字符串。
flags   标志位，用于控制正则表达式的匹配方式，如：是否区分大小写，多行匹配等等。

功能：在string中进行搜索，成功返回Match object, 失败返回None, 只匹配一个。

## findall

在string中查找所有匹配成功的组,即用括号括起来的部分。返回list对象，每个list item是由每个匹配的所有组组成的list。

## finditer

在string中查找所有匹配成功的字符串,返回iterator，每个item是一个Match object。

## 特殊字符

[Python 正则表达式](https://www.runoob.com/python/python-reg-expressions.html)

## 样例代码

~~~python
import re

content = '333STR1666STR299'
regex = r'([A-Z]+(\d))'

if __name__ == '__main__':
    print(re.match(regex, content)) ##content的开头不符合正则，所以结果为None。

    ##只会找一个匹配，match[0]是regex所代表的整个字符串，match[1]是第一个()中的内容，match[2]是第二对()中的内容。
    match = re.search(regex, content)
    print('\nre.search() return value: ' + str(type(match)))
    print(match.group(0), match.group(1), match.group(2))  

    result1 = re.findall(regex, content)
    print('\nre.findall() return value: ' + str(type(result1)))
    for m in result1:
        print(m[0], m[1])

    result2 = re.finditer(regex, content)
    print('\nre.finditer() return value: ' + str(type(result2)))
    for m in result2:
        print(m.group(0), m.group(1), m.group(2))  ##字符串
~~~

## 运行结果

~~~
None 

re.search() return value: <TYPE ?_sre.SRE_Match?>
STR1 STR1 1 

re.findall() return value: <TYPE ?list?>
STR1 1 STR2 2 

re.finditer() return value: <TYPE ?callable-iterator?>
STR1 STR1 1 
STR2 STR2 2
~~~


# excel 中的应用

参考博客：[传送门](https://www.cnblogs.com/insane-Mr-Li/p/9092619.html)

python操作excel主要用到xlrd和xlwt这两个库，即xlrd是读excel，xlwt是写excel的库。

## xlrd

~~~python
data = xlrd.open_workbook(filename)#文件名以及路径，如果路径或者文件名有中文给前面加一个r拜师原生字符。
~~~

1）获取book中一个工作表

~~~python
table = data.sheets()[0]          #通过索引顺序获取

table = data.sheet_by_index(sheet_indx)) #通过索引顺序获取

table = data.sheet_by_name(sheet_name)#通过名称获取

以上三个函数都会返回一个xlrd.sheet.Sheet()对象

names = data.sheet_names()    #返回book中所有工作表的名字

data.sheet_loaded(sheet_name or indx)   # 检查某个sheet是否导入完毕
~~~

2）行的操作

~~~python
nrows = table.nrows  #获取该sheet中的有效行数

table.row(rowx)  #返回由该行中所有的单元格对象组成的列表

table.row_slice(rowx)  #返回由该列中所有的单元格对象组成的列表

table.row_types(rowx, start_colx=0, end_colx=None)    #返回由该行中所有单元格的数据类型组成的列表

table.row_values(rowx, start_colx=0, end_colx=None)   #返回由该行中所有单元格的数据组成的列表

table.row_len(rowx) #返回该列的有效单元格长度
~~~

3）列(colnum)的操作

~~~python
ncols = table.ncols   #获取列表的有效列数

table.col(colx, start_rowx=0, end_rowx=None)  #返回由该列中所有的单元格对象组成的列表

table.col_slice(colx, start_rowx=0, end_rowx=None)  #返回由该列中所有的单元格对象组成的列表

table.col_types(colx, start_rowx=0, end_rowx=None)    #返回由该列中所有单元格的数据类型组成的列表

table.col_values(colx, start_rowx=0, end_rowx=None)   #返回由该列中所有单元格的数据组成的列表
~~~

4）单元格的操作  

~~~python
table.cell(rowx,colx)   #返回单元格对象

table.cell_type(rowx,colx)    #返回单元格中的数据类型

table.cell_value(rowx,colx)   #返回单元格中的数据

table.cell_xf_index(rowx, colx)   # 暂时还没有搞懂
~~~

## xlwt

xlwt就那么四五个函数，都在这段代码里了。这段代码就差格式相关的内容了，需要的话自己去查[API](https://xlwt.readthedocs.io/en/latest/api.html)。


~~~python
import xlwt
import os

# excel的存放路径
excel_path = r'C:\Users\zd\Desktop\test.xls'

class ExcelWrite(object):
    def __init__(self):
        self.excel = xlwt.Workbook()  # 创建一个工作簿
        self.sheet = self.excel.add_sheet('Sheet1')  # 创建一个工作表
    
    # 写入单个值
    def write_value(self, cell, value):
        '''
            - cell: 传入一个单元格坐标参数，例如：cell=(0,0),表示修改第一行第一列
        '''
        self.sheet.write(*cell, value)
        # （覆盖写入）要先用remove(),移动到指定路径，不然第二次在同一个路径保存会报错
        os.remove(excel_path)
        self.excel.save(excel_path)

~~~~



# logging


~~~python
logging.basicConfig(level = logging.ERROR,format = '%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
~~~~

# 异常

~~~python
try:

　　...

except Exception as e:

　　...
~~~~

# queue

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。

Queue.qsize() 返回队列的大小
Queue.empty() 如果队列为空，返回True,反之False
Queue.full() 如果队列满了，返回True,反之False，Queue.full 与 maxsize 大小对应
Queue.get([block[, timeout]])获取队列，timeout等待时间
Queue.get_nowait() 相当于Queue.get(False)，非阻塞方法
Queue.put(item) 写入队列，timeout等待时间
Queue.task_done() 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号。每个get()调用得到一个任务，接下来task_done()调用告诉队列该任务已经处理完毕。
Queue.join() 实际上意味着等到队列为空，再执行别的操作


# 与linux交互


~~~python
a=os.system('ls')   ##得到的是执行的命令的返回值，并不是执行结果。成功，为0
# a:
# 0

b=os.popen('ls').readlines()    #将得到的结果直接赋值给b列表
# b：
# ['anaconda-ks.cfg\n',
#  'epel-release-7-5.noarch.rpm\n',
#  'ipython-4.1.2\n',
#  'ipython-4.1.2.tar.gz\n',
#  'pip-8.1.2\n',
#  'pip-8.1.2.tar.gz#md5=87083c0b9867963b29f7aba3613e8f4a.gz\n']
~~~~

# 实用内置函数

## Queue

~~~py
Queue.qsize() # 返回队列的大小
Queue.empty() # 如果队列为空，返回True,反之False
Queue.full() # 如果队列满了，返回True,反之False，Queue.full 与 maxsize 大小对应
Queue.get([block[, timeout]])# 获取队列，timeout等待时间
Queue.get_nowait() # 相当于Queue.get(False)，非阻塞方法
Queue.put(item) # 写入队列，timeout等待时间
Queue.task_done() # 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号。每个get()调用得到一个任务，接下来task_done()调用告诉队列该任务已经处理完毕。
Queue.join() # 实际上意味着等到队列为空，再执行别的操作
~~~~

## numpy

### 创建及初始化

~~~python
a = np.array([2,3,4])
b = np.array([2.0,3.0,4.0])
c = np.array([[1.0,2.0],[3.0,4.0]])
d = np.array([[1,2],[3,4]], dtype = complex) # 指定数据类型
print np.arange(0,7,1,dtype = np.int16) # 0为起点，间隔为1时可缺省(引起歧义下不可缺省)
print np.ones((2,3,4),dtype = np.int16) # 2页，3行，4列，全1，指定数据类型
print np.zeros((2,3,4)) # 2页，3行，4列，全0
print np.empty((2,3)) #值取决于内存
print np.arange(0,10,2) # 起点为0，不超过10，步长为2
print np.linspace(-1,2,5) # 起点为-1，终点为2，取5个点
print np.random.randint(0,3,(2,3)) # 大于等于0，小于3，2行3列的随机整数
~~~~

### 修改

使用 np.c_[] 和 np.r_[] 分别添加行和列

~~~python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])
b = np.ones(3)
np.c_[a,b]
array([[ 1.,  2.,  3.,  1.],
    [ 4.,  5.,  6.,  1.],
    [ 7.,  8.,  9.,  1.]])
~~~~

### shape
~~~py
x = np.array([[1,2,5],[2,3,5],[3,4,5],[2,3,6]])
#输出数组的行和列数
print x.shape  #结果： (4, 3)
#只输出行数
print x.shape[0] #结果： 4
#只输出列数
print x.shape[1] #结果： 3
~~~~

### 内置函数
~~~python
~~~~

## 纬度范围表示

[1:]   意思是去掉列表中第一个元素（下标为0），去后面的元素进行操作，以一个示例题为例，用在遍历中统计个数：
[::-1]  从最后一个元素到第一个元素复制一遍
a[i:j] 表示复制a[i]到a[j-1]
a[i:j:s] 表示复制a[i]到a[j-1]，但s表示步进，缺省为1.

## counter

Counter 集成于 dict 类，因此也可以使用字典的方法，此类返回一个以元素为 key 、元素个数为 value 的 Counter 对象集合

~~~py
import collections
obj = collections.Counter('aabbccc')
print(obj)

#输出：Counter({'c': 3, 'a': 2, 'b': 2})

~~~~


## 路径

~~~py
for root, dirs, files in os.walk(data_path):
    # print(root)
    book_name = re.search('第.+册', root)
    if not book_name:
      continue
    book_name = book_name.group()
    for file in files:
      file_path = os.path.join(root, file)

os.listdir()  # 获取路径下的所有文件、文件夹名。不递归
~~~~

每次遍历的对象都是返回的是一个三元组(root,dirs,files)

root 所指的是当前正在遍历的这个文件夹的本身的地址
dirs 是一个 list ，内容是该文件夹中所有的目录的名字(不包括子目录)
files 同样是 list , 内容是该文件夹中所有的文件(不包括子目录)

~~~py
os.path.exists(path) # 判断一个目录或文件是否存在

os.makedirs(path) # 多层创建目录

os.mkdir(path) # 创建目录
~~~~


## 时间

~~~py
import datetime

start = datetime.datetime.now()
for i in range(1000):
    for j in range(500):
        m = i + j
        print(m)
end = datetime.datetime.now()
print(end - start)
# 0:00:03.952226
~~~~

## defaultdict

~~~py
from collections import defaultdict
dict1 = defaultdict(int)
~~~

## 其他

### 内存相关

~~~python

import psutil
import os

def memory_usage():
  mem_available = psutil.virtual_memory().available
  mem_process = psutil.Process(os.getpid()).memory_info().rss
  return round(mem_process / 1024 / 1024, 2), round(mem_available / 1024 / 1024, 2)

print(memory_usage(), flush = True)
a = np.random.rand(100, 1024, 1024)
print(memory_usage())
~~~~

### enumerate 函数

~~~python
enumerate(sequence, [start=0])
~~~~

sequence -- 一个序列、迭代器或其他支持迭代对象。
start -- 下标起始位置


~~~python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons))
#[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
list(enumerate(seasons, start=1))       # 下标从 1 开始
#[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]

seq = ['one', 'two', 'three']
for i, element in enumerate(seq):
    print i, element

#0 one
#1 two
#2 three
~~~~

### 边边角角

int最大最小值：

~~~python
import sys
MAX_INT = sys.maxsize
print(MAX_INT)
print(-sys.maxint - 1)

max_float=float('inf')
print(max_float)
~~~~




# 高级语法

## 列表生成式的工作过程

格式： [experssion for i in sequence if...]

工作过程：

  迭代序列sequence中的每个元素，每次迭代都先判断if表达式结果为真，如果为真则进行下一步，如果为假则进行下一次迭代；

  把迭代结果赋值给i，然后通过experssion得到一个新的计算值；

  最后把所有通过experssion得到的计算值以一个新列表的形式返回。

例：找出1~num中所有的质数

~~~python

def isPrime(num):
  for i in range(2, num):
    if num & i == 0:
      return False
  return True

print [i for i in range(1, num + 1) if isPrime(i)]
~~~~

## 字典生成式

生成20个学生并且筛选出成绩在90-100的学生。

~~~python
stu = {str(i): random.randint(60, 100) for i in range(20)}
res = {name: score for name, score in stu.items() if score > 90}
print(res)
~~~~