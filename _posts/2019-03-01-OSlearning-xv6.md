---
layout: post
title: "OS learning"
description: "个人学习OS笔记（1）xv6"
categories: [OS]
tags: [OS]
redirect_from:
  - /2019/03/01/
---

* Karmdown table of content
{:toc .toc}

# 下载地址

[linux 14.04，32位](http://old-releases.ubuntu.com/releases/14.04.3/)

[xv6](https://github.com/mit-pdos/xv6-public)

下载后截图：
图1

# 源码概览

xv6文件目录嵌套并不是很多。可以通过阅读Makefile文件来理清文件的关系。也可以查阅相关博客进行学习。

在xv6中，由16位和32位汇编混合编写而成的bootasm.S，和一个由 C 写成的bootmain.c组成[bootloader](https://en.wikibooks.org/wiki/X86_Assembly/Bootloaders)。简单点说，bootloader就是一段代码，而启动一个系统，假设不考虑硬件本身上电触发的底层程序，操作系统的入口程序就是bootloader。

