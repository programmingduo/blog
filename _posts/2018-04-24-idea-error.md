---
layout: post
title: "idea error"
description: "idea 显示无法找到或加载主类"
categories: [environment]
tags: [windows]
redirect_from:
  - /2018/04/24/
---

* Karmdown table of content
{:toc .toc}

使用idea开发的过程中，第一次跑代码正常运行，后来打开后就显示：
找不到或无法加载主类

在网上查了一些资料，无非就是几种处理方式：

1：
idea本身缓存出问题。解决方法：
file->invalidate Cache/restart

2：
输出路径设置有问题。解决方法：[参考博客](https://blog.csdn.net/qq_31382921/article/details/72898134)

3：
创建项目的时候出了问题。解决方法：[参考博客](https://blog.csdn.net/gxx_csdn/article/details/79059884)