---
layout: post
title: "maven set up"
description: "maven set up"
categories: [environment]
tags: [windows]
redirect_from:
  - /2018/04/09/
---

* Karmdown table of content
{:toc .toc}

首先需要jdk


下载[source](http://maven.apache.org/download.cgi)

![smiley](\assets\images\usedInBlogs\maven\1.png)

下载Binary zip archive

下载下来之后，解压，找个路径放进去， 

把bin的位置设在环境变量里，新建环境变量MAVEN_HOME

![smiley](\assets\images\usedInBlogs\maven\2.png)

在PATH里加入maven的bin的路径

![smiley](\assets\images\usedInBlogs\maven\3.png)

配置完毕后，在Windows命令提示符下，输入mvn -v测试一下，配置成功显示如图：
![smiley](\assets\images\usedInBlogs\maven\4.png)