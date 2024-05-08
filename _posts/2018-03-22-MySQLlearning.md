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




# 检索

~~~mysql
SELECT DISTINCT uesr FROM menu
SELECT user FROM menu LIMIT 4
SELECT user FROM menu LIMIT 3 OFFSET 4

~~~~

# 排序

~~~ruby
SELECT prob_name FROM product
SELECT prob_name FROM product ORDER BY prob_name
SELECT prob_id, prob_price, prob_name FROM product ORDER BY prod_price DESC, prob_name
SELECT prob_price FROM product ORDER BY prob_price DESC LIMIT 1
~~~~

# 过滤

* 在同时使用where和order by子句时，应该让order by位于where之后，否则将会产生错误


~~~ruby
SELECT prod_name, prod_price FROM product WHERE prod_price BETWEEN 5 AND 10
# between 匹配范围中的所有值，包括指定的开始值和结束值

SELECT prod_name, prod_price FROM product WHERE vend_id = 1002 OR vend_id=1003 AND prod_price=10
# 效果等于  WHERE vend_id = 1002 OR (vend_id=1003 AND prod_price=10)
# SQL与大多数编程语言一样，处理OR操作符前，会优先处理and操作符

SELECT prod_name, prod_price FROM product WHERE vend_id NOT IN (1002, 1003) ORDER BY prod_name
~~~~


## 通配符

通配符搜索的处理时间比普通的过滤要花更多的时间

~~~ruby
SELECT prob_id FROM product WHERE prob_name LIKE "%000"
# %可以匹配0个、1个、多个任意字符

SELECT prob_id, prob_name FROM product WHERE prob_name LIKE "_ ton anvil"
# _匹配单个任意字符
~~~~

## 正则

mysql的正则是对列值内进行匹配，而like是对整个列值匹配
~~~ruby
SELECT prob_id FROM product WHERE prob_name REGEXP "1000"
# ”Jet 1000“也会返回
~~~

mysql默认不匹配大小写，如果要区分大小写，可使用BINARY关键字。如 “WHERE prob_name REGEXP BINARY 'JetPack .000'”。

~~~ruby
 SELECT prob_name FROM product WHERE prob_name REGEXP '1000|2000'
 SELECT prob_name FROM product WHERE prob_name REGEXP '[123]000'
 SELECT prob_name FROM product WHERE prob_name REGEXP '[1-3]000'
 SELECT prob_name FROM product WHERE prob_name REGEXP '[^123]000'
 # 上述[^123]语句匹配除123外的任意字符
 ~~~

mysql 中匹配特殊字符需要使用两个反斜杠字符，一个用于mysql本身，一个用于正则表达式。

~~~ruby
 SELECT prob_name FROM product WHERE prob_name REGEXP '\\.'
 # 上述语句匹配特殊字符'.'
~~~

mysql 提供正则的简单测试语句
~~~ruby
SELECT 'hell0' REGEXP '[0-9]'
# 如果返回1，则表示匹配，返回0则表示不匹配
~~~~


# 创建计算字段

mysql可以使用Concat()将字符进行拼接，使用RTrim删除值右边的所有空格，LRtrim()删除值左边的所有空格，Trim()删除左右两边的空格。

mysql使用AS 来标识别名(alias)

~~~ruby
SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country), ')') AS vend_title
FROM vendors
ORDER BY vend_name;
~~~

~~~ruby
SELECT prob_id, quantity, item_price, quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
~~~~


# 函数




# 日期相关
直接上样例代码。

~~~ruby
SELECT * FROM product WHERE Date(add_time) = '2013-01-12';
SELECT * FROM product WHERE Date(add_time) between '2013-01-01' and '2013-01-31';
SELECT * FROM product WHERE Year(add_time) = 2013 and Month(add_time) = 1;
SELECT something FROM table WHERE TO_DAYS(NOW()) - TO_DAYS(date_col) <= 30;
~~~

详见 <http://blog.csdn.net/qq2712193/article/details/48766575>

