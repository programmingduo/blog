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

# 输入输出

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
print ("test", file=f)
f.close
~~~~~~~~~

open函数可以打开或者创建文件，第二个参数是文件权限。w表示写，a表示append，拓展。

如果参数为"w"每次打开文件都会把文件原有内容删除。

而若参数为"a"则会在文件尾部添加新内容。


# 正则匹配

re就Python中用于正则表达式相关处理的类，这四个方法都是用于匹配字符串的，具体区别如下：

## match

匹配string 开头，成功返回Match object, 失败返回None，只匹配一个。

## search

在string中进行搜索，成功返回Match object, 失败返回None, 只匹配一个。

## findall

在string中查找所有匹配成功的组,即用括号括起来的部分。返回list对象，每个list item是由每个匹配的所有组组成的list。

## finditer

在string中查找所有匹配成功的字符串,返回iterator，每个item是一个Match object。

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



