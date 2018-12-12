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

![smiley](\assets\images\usedInBlogs\golang\1.jpg)

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

[地址](http://www.runoob.com/go/go-tutorial.html)。这里只贴一些笔记。

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

![smiley](\assets\images\usedInBlogs\golang\2.jpg)

## 函数

在go中，定义函数的方法如下：

~~~go
func (p myType ) funcName ( a, b int , c string ) ( r , s int ) {
    return
}
~~~~

与其他语言对比，他们的共性有：
> 关键字——func
>
> 方法名——funcName
>
> 入参——— a,b int,b string
>
> 返回值—— r,s int
>
> 函数体—— {}

Go中函数的特性：

> 为特定类型定义函数，即为类型对象定义方法
>
> 在Go中通过给函数标明所属类型，来给该类型定义方法，上面的"p myType"即表示给myType声明了一个方法，"p myType"不是必须的。如果没有，则纯粹是一个函数，通过包名称访问。packageName.funcationName

如：

~~~~~go

//定义新的类型double，主要目的是给float64类型扩充方法
type double float64

//判断a是否等于b
func (a double) IsEqual(b double) bool {
    var r = a - b
    if r == 0.0 {
        return true
    } else if r < 0.0 {
        return r > -0.0001
    }
    return r < 0.0001
}

//判断a是否等于b
func IsEqual(a, b float64) bool {
    var r = a - b
    if r == 0.0 {
        return true
    } else if r < 0.0 {
        return r > -0.0001
    }
    return r < 0.0001
}

func main() {
    var a double = 1.999999
    var b double = 1.9999998
    fmt.Println(a.IsEqual(b))
    fmt.Println(a.IsEqual(3))
    fmt.Println( IsEqual( (float64)(a), (float64)(b) ) )

}

~~~~~

上述示例为 float64 基本类型扩充了方法IsEqual，该方法主要是解决精度问题。其方法调用方式为： a.IsEqual(double) ，如果不扩充方法，我们只能使用函数IsEqual(a, b float64)

入参中，如果连续的参数类型一致，则可以省略连续多个参数的类型，只保留最后一个类型声明。
如 func IsEqual(a, b float64) bool 这个方法就只保留了一个类型声明,此时入参a和b均是float64数据类型。 这样也是可以的： func IsEqual(a, b float64, accuracy int) bool

变参：入参支持变参,即可接受不确定数量的同一类型的参数
如 func Sum(args ...int) 参数args是的slice，其元素类型为int 。经常使用的fmt.Printf就是一个接受任意个数参数的函数 fmt.Printf(format string, args ...interface{})

支持多返回值
前面我们定义函数时返回值有两个r,s 。这是非常有用的，我在写C#代码时，常常为了从已有函数中获得更多的信息，需要修改函数签名，使用out ,ref 等方式去获得更多返回结果。而现在使用Go时则很简单，直接在返回值后面添加返回参数即可。

函数部分的讲解详见：[go语言中文网](https://studygolang.com/articles/7529)

## nil

在Go语言中，如果你声明了一个变量但是没有对它进行赋值操作，那么这个变量就会有一个类型的默认零值。

这是每种类型对应的零值：

> bool      -> false                              
> 
> numbers -> 0                                 
> 
> string    -> ""      
> 
> slices -> nil
> 
> pointers -> nil
> 
> maps -> nil
> 
> channels -> nil
> 
> functions -> nil
> 
> interfaces -> nil

举个例子，当你定义了一个struct：

~~~go
type Person struct {
  AgeYears int
  Name string
  Friends []Person
}

var p Person // Person{0, "", nil}
~~~~

详见于[go语言中文网](https://studygolang.com/articles/9506)

## boltdb

详见于 [go语言中文网](https://studygolang.com/articles/10433)

对自己英语水平有自信的可以直接读[官网wiki](https://github.com/boltdb/bolt)