---
layout: post
title: "golang learning"
description: "golang个人学习笔记"
categories: [language]
tags: [golang,windows]
redirect_from:
  - /2018/05/03/
---

* Karmdown table of content
{:toc .toc}

# 一、环境配置
这个是我见过最好配置的一门语言了。没有之一。

[下载传送门](https://www.golangtc.com/download)

图片：1

下载之后双击运行。然后，就好了。。。。

# 二、hello world！

## 单个文件

这个也是为了测试安装环境的。

文件名：test.go

代码如下：

~~~golang
package main

import "fmt"

func main() {
   fmt.Println("Hello, World!")
}
~~~~

在相应路径下使用命令：

~~~
go run test.go
~~~~

即可看到正常输出。

## 项目工程

需要使用命令行先进编译生成.exe文件。之后运行.exe文件。

编译命令：

~~~~
go build
~~~~~

默认情况是你的package名(非main包)，或者是第一个源文件的文件名(main包)。

你也可以指定编译输出的文件名命令：

~~~~
go build -o astaxie.exe
~~~~~~




# 语法及教程

http://www.runoob.com/go/go-tutorial.html。这里只贴一些笔记。

## 变量

样例

~~~golang
package main

var x, y int
var (  // 这种因式分解关键字的写法一般用于声明全局变量
    a int
    b bool
)

var c, d int = 1, 2
var e, f = 123, "hello"

//这种不带声明格式的只能在函数体中出现
//g, h := 123, "hello"

func main(){
    g, h := 123, "hello"
    println(x, y, a, b, c, d, e, f, g, h)
}
~~~~~

输出

*  0 0 0 false 1 2 123 hello 123 hello

图片：2



学习

