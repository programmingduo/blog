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

# 基础基础之基础

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

# 输入输出

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


# 字符串处理与正则匹配

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

## split

~~~python
str.split(str="", num=string.count(str)).
~~~~

参数：

> str -- 分隔符，默认为所有的空字符，包括空格、换行(\n)、制表符(\t)等。
> 
> num -- 分割次数。默认为 -1, 即分隔所有。

re就Python中用于正则表达式相关处理的类，这四个方法都是用于匹配字符串的，具体区别如下：

## match

匹配string 开头，成功返回Match object, 失败返回None，只匹配一个。


## search

函数原型：

~~~
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

https://www.runoob.com/python/python-reg-expressions.html

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

# 实用内置函数

## numpy

### 创建

~~~python
a = np.array([2,3,4])
b = np.array([2.0,3.0,4.0])
c = np.array([[1.0,2.0],[3.0,4.0]])
d = np.array([[1,2],[3,4]], dtype = complex) # 指定数据类型
~~~~

### 内置函数
~~~python
print np.arange(0,7,1,dtype = np.int16) # 0为起点，间隔为1时可缺省(引起歧义下不可缺省)
print np.ones((2,3,4),dtype = np.int16) # 2页，3行，4列，全1，指定数据类型
print np.zeros((2,3,4)) # 2页，3行，4列，全0
print np.empty((2,3)) #值取决于内存
print np.arange(0,10,2) # 起点为0，不超过10，步长为2
print np.linspace(-1,2,5) # 起点为-1，终点为2，取5个点
print np.random.randint(0,3,(2,3)) # 大于等于0，小于3，2行3列的随机整数
~~~~

使用 np.c_[] 和 np.r_[] 分别添加行和列

~~~python
a = np.array([[1,2,3],[4,5,6],[7,8,9]])
b = np.ones(3)
np.c_[a,b]
array([[ 1.,  2.,  3.,  1.],
    [ 4.,  5.,  6.,  1.],
    [ 7.,  8.,  9.,  1.]])
~~~~

~~~py

x = np.array([[1,2,5],[2,3,5],[3,4,5],[2,3,6]])
#输出数组的行和列数
print x.shape  #结果： (4, 3)
#只输出行数
print x.shape[0] #结果： 4
#只输出列数
print x.shape[1] #结果： 3

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

## 其他

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
