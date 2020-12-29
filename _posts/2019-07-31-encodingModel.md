---
layout: post
title: "encoding model"
description: "encoding model"
categories: [NLP]
tags: [encoding model]
redirect_from:
  - /2019/07/31/
---

* Karmdown table of content
{:toc .toc}

# Stacked AutoEncoder(SAE)

参考博客：[Stacked AutoEncoder（SAE）堆栈自编码器](https://blog.csdn.net/qq_41319343/article/details/83997862)。这里近做一些摘录。

1.png

也是通过构建输出x与输入x尽可能相似的网络来进行无监督学习。使用均方误差来衡量相似度，因此模型的目标为最小化均方重构误差。

在第一层AE训练好之后第一层的隐层输出结果作为第二层的输入传入到第二层。stacked也就是逐层堆叠的意思。

比较奇特的一点是，常见的神经网络都是一次迭代（一个样本或一个batch），更新所有层的参数。而SAE则是所有样本迭代一次，仅更新某一层，逐层训练。