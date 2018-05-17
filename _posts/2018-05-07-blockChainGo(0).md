---
layout: post
title: "block chain go (0)"
description: "区块链和比特币学习博客"
categories: [block-chains,bit-coins]
tags: [block-chains,bit-coins,golang]
redirect_from:
  - /2018/05/07/
---

* Karmdown table of content
{:toc .toc}

在网上看到有人使用go语言实现了简单的区块链。突然感到很有兴趣，决定按照教程也实现一个区块链。

原文链接：

[英文](https://jeiwan.cc/tags/blockchain/)

[中文](https://blockchain.liuchengxu.org/part-1/basic-prototype.html)

中文链接并不是很稳定。有时候会莫名其妙无法打开。遂亲自翻译英文版教程于本人博客。详情见后续博客。

在本系列博客中除了进行翻译，还加入了整个项目的文件管理和项目搭建工作。（所有go文件中package均为main，import自行添加，后续文章不再赘述）

在使用go实现区块链前建议阅读一下两篇博客以对于区块链有初步认识。
[block chain](http://wuduo.me/blog/2018/04/26/block-chain/)
[bit coin](http://wuduo.me/blog/2018/04/28/bit-coins/)

另外，这里贴上两个golang相关的网址，变成过程中如遇到语言问题可进行查阅：
[golang doc](https://golang.org/doc/)
[golang China](https://studygolang.com/)

此教程使用代码可见于[本人github](https://github.com/programmingduo/BlockChain)

也可参考[另一github](https://github.com/Jeiwan/blockchain_go/tree/master)，该项目的优点在于，可以选择分支来查看每一章节的代码。