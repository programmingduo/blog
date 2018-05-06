---
layout: post
title: "markdown 语法与本博客实现"
description: "简单易懂的入门markdown语法以及本模板中的语法"
categories: [language]
tags: [markdown]
redirect_from:
  - /2013/04/22/
---

参考博客：<http://www.appinn.com/markdown>

里边有较为全面的讲解。这里列举的是本模板中的格式及语法（部分通用）。

* Karmdown table of content
{:toc .toc}

# 标题

以上“标题”二字便是通过代码：
>\#标题

实现的。
个人比较喜欢的格式：
>\# 这是 H1
> 
>\## 这是 H2
>
>\###### 这是 H6

# 引用

引用主要使用到的是： “\>”
本模板中有highlight，故引用使用的地方并不多。

# 博客title问题
"_posts"路径下的文件名并非博客的title，而是浏览网站时的URL。

博客的title是根据博客里面的title一项信息定的。

# 博客路径问题
博客中使用的路径从博客的根路径开始，并无其他配置。