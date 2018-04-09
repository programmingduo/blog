---
layout: post
title: "CHECKERFRAMEWORK"
description: "配置CHECKERFRAMEWORK"
categories: [tools]
tags: [CheckerFrameWork]
redirect_from:
  - /2018/03/23/
---

* Karmdown table of content
{:toc .toc}

# 开发手册
<https://checkerframework.org>

# 环境安装与配置
[下载](https://checkerframework.org/checker-framework-2.4.0.zip) 并解压。

添加系统变量

变量名：CHECKERFRAMEWORK

变量值：C:\Program Files\checker-framework-2.4.0\

在cmd中执行以下命令：

~~~~
doskey javacheck=java -jar "%CHECKERFRAMEWORK%\checker\dist\checker.jar" $*
~~~~~~

然后使用javacheck执行java文件即可。如：

~~~~
javacheck .\frame.java
~~~~~~

# 测试maven项目样例
checker ramework中自带maven样例。在docs/examples/MavenExample/ 下。
导入eclipse的maven项目之后，构建过程会出现以下提示。

~~~~~
Plugin execution not covered by lifecycle configuration: org.apache.maven.plugins:maven-dependency-plugin:2.8:properties (execution: default, phase: initialize)
~~~~~~~

解决办法：
 <plugins> 标签外再套一个 <pluginManagement> 标签

 亲测好使。