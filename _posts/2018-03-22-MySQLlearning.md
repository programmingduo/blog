---
layout: post
title: "MySQL learning"
description: "个人学习Python笔记"
categories: [language]
tags: [MySql]
redirect_from:
  - /2018/03/22/
---

* Karmdown table of content
{:toc .toc}

# 日期相关
直接上样例代码。

~~~ruby
SELECT * FROM product WHERE Date(add_time) = '2013-01-12';
SELECT * FROM product WHERE Date(add_time) between '2013-01-01' and '2013-01-31';
SELECT * FROM product WHERE Year(add_time) = 2013 and Month(add_time) = 1;
SELECT something FROM table WHERE TO_DAYS(NOW()) - TO_DAYS(date_col) <= 30;
~~~

详见 <http://blog.csdn.net/qq2712193/article/details/48766575>