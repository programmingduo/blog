---
layout: post
title: "block chain go (2)：Proof-of-Work"
description: "区块链和比特币学习博客"
categories: [block-chains]
tags: [block-chains,golang]
redirect_from:
  - /2018/05/07/
---

* Karmdown table of content
{:toc .toc}

# 原文链接
[Building Blockchain in Go. Part 2: Proof-of-Work](https://jeiwan.cc/posts/building-blockchain-in-go-part-2/)

# 简介

在[上一章]()我们已经建立了一个简单的数据结构，实现了区块链这一数据库最基础、本质、核心的部分。我们已经可以通过链式关系增加区块。但是这一实现有一个显著的问题：增加区块十分的简单并且廉价。区块链以及比特币的一个很重要的原则便是增加区块是一个很复杂的事情。这一篇博客我们将修复这一缺陷。

# 工作量的证明

区块链的一个很关键的思想便是在向其存储数据的时候必须先经过大量的工作。正是这些工作保证了区块链的安全和一致。此外，这些工作也会得到一些报酬(也就是矿工在挖矿的时候获得的报酬)

这种机制跟现实生活中的情形十分相似：一个人必须努力工作来获得薪酬以维持生活。在区块链中，一些参与者(矿工)通过工作来维持这一网络，为该网络增加区块并且获得酬劳。他们的工作可以使新的区块安全的加入区块链当中，这也保证了整个区块链数据库的安全性和稳定性，保证整个区块链可以不断成长。值得注意的是，矿工必须证明自己完成了他的工作。

整个“完成工作并证明已完成”机制被称为“工作量的证明”(原文："[ proof-of-work](https://en.wikipedia.org/wiki/Proof-of-work_system)")。这个过程需要巨大的计算量（即使是高性能的机器也很难在短时间内完成它）使得该过程十分困难。此外，这一过程也会随着时间而变得越来越困难，以保证每小时增加六个区块的速度。

在比特币中，这一工作的目标是找到区块的哈希值，这一哈希值必须满足一些特定的需要。并且这一哈希值用来完成证明。因此，寻找证明（有效哈希）便是矿工实际的工作。

还有一点需要注意。“工作量的证明”这一机制必须满足一个要求：完成工作十分困难而证明过程比较简单。证明（找到的哈希值）通常被交给其他人，所以对于这些人而言，核实过程不应当花费太多时间。

# 哈希

这一节我们将讨论哈希问题。如果你熟悉这一部分，你可以选择跳过。

所谓哈希：
![smiley](\assets\images\usedInBlogs\blockChainGo\2\1.png)

* 任意长度的数据经过哈希都可以得到固定长度的哈希值。
* 哈希不是加密，我们无法从哈希值得到原来的数据。
* 特定的数据只能生成唯一的哈希值，这一哈希值也只能由该数据生成。
* 改变数据的任一字符都会得到一个完全不同的哈希值。

哈希被广泛用于检测数据是否一致。一些软件供应商会在软件里提供"总和检查"。在下载软件之后可以生成哈希值，跟供应商提供的哈希值比较之后便可得知是否正确下载。

在区块量当中，哈希被用于保证区块的一致性。哈希算法的输入数据中包含前一区块的哈希值，这保证了改变区块链中的任一区块都是不可能的(至少十分困难)————他必须重新计算要更改区块的哈希值并且重新计算这一区块后面的所有区块的哈希值。

# Hashcash

比特币使用[Hashcash](https://en.wikipedia.org/wiki/Hashcash)，一个最初用来防止垃圾邮件的工作量证明算法。它可以被分解为以下步骤：

* 取一些公开的数据（比如，如果是email的话，它可以是接收者的邮件地址；在比特币中，它是区块头）
* 给这个公开数据添加一个计数器。计数器默认从 0 开始
* 将 data(数据) 和 counter(计数器) 组合到一起，获得一个哈希
* 检查哈希是否符合一定的条件：
  * 如果符合条件，结束
  * 如果不符合，增加计数器，重复步骤 3-4

因此，这是一个暴力算法：改变计数器，计算新的哈希，检查，增加计数器，计算哈希，检查，如此往复。这也是为什么说它的计算成本很高。

现在，让我们来认真看一下一个哈希要满足的必要条件。在原来的 Hashcash 实现中，它的要求是 “一个哈希的前 20 位必须是‘0’”。在比特币中，这个要求会随着时间而不断变化。因为按照设计，不论计算能力随着时间增长，或者是越来越多的矿工进入网络，都必须保证每 10分钟生成一个块。所以需要动态调整这个必要条件。

为了展示这一算法，我从前一个例子（“I like donuts”）中取得数据，并且找到了一个前 3 个字节是全是 0 的哈希。

![smiley](\assets\images\usedInBlogs\blockChainGo\2\2.png)

ca07ca 是十进制值 13240266.的十六进制表示。

# 实现
好了，终于完成了理论层面（翻译真的是一个困难的过程），来动手写代码吧！首先，定义挖矿的难度值：

(新建文件 proofOfWork.go)
~~~~go
const targetBits = 24
~~~~~~

在比特币中，当一个块被挖出来以后，“target bits” 代表了区块头里存储的难度，也就是开头有多少个 0。这里的 24 指的是算出来的哈希前 24 位必须是 0，如果用 16 进制表示，就是前 6 位必须是 0，这一点从最后的输出可以看出来。目前我们并不会实现一个动态调整目标的算法，所以将难度定义为一个全局的常量即可。

24 其实是一个可以任意取的数字，其目的只是为了有一个小于256位的目标而已。同时，我们需要该目标值有明显的意义（确实让矿工的工作量加大了），但是又不至于大的过分（我们只是在自己的计算机里模拟，计算量太大并不利于学习过程）。难度值（这里设为24）越大，则越难找到合适的区块。

~~~~go
type ProofOfWork struct {
	block  *Block
	target *big.Int
}

func NewProofOfWork(b *Block) *ProofOfWork {
	target := big.NewInt(1)
	target.Lsh(target, uint(256-targetBits))
	//Lsh: local sensitivity hashing
	//将1左移232（256-24）位

	pow := &ProofOfWork{b, target}

	return pow
}
~~~~~

参考网址：
[lsh](https://en.wikipedia.org/wiki/Locality-sensitive_hashing)
[big](https://golang.org/pkg/math/big/)

这里，我们构造了 ProofOfWork 结构，里面存储了指向一个块(block)和一个目标(target)的指针。这里的 “目标” ，也就是前一节中所描述的难度。这里使用了一个 大整数 ，我们会将哈希与目标进行比较：先把哈希转换成一个大整数，然后检测它是否小于目标。

在 NewProofOfWork 函数中，我们将 big.Int 初始化为 1，然后左移 256 - targetBits 位。256 是一个 SHA-256 哈希的位数，我们将要使用的是 SHA-256 哈希算法。target（目标） 的 16 进制形式为：

> 0x10000000000000000000000000000000000000000000000000000000000

它在内存上占据了 29 个字节。下面是与前面例子哈希的形式化比较：

> 0fac49161af82ed938add1d8725835cc123a1a87b1b196488360e58d4bfb51e3
> 
> 0000010000000000000000000000000000000000000000000000000000000000
> 
> 0000008b0f41ec78bab747864db66bcb9fb89920ee75f43fdaaeb5544f7f76ca

第一个哈希（基于“I like donuts”计算）比目标要大，因此它并不是一个有效的工作量证明。第二个哈希（基于 “I like donutsca07ca” 计算）比目标要小，所以是一个有效的证明。

你可以把目标想象为一个范围的上界：如果一个数（由哈希转换而来）比上界要小，那么是有效的，反之无效。因为要求比上界要小，所以会导致有效数字并不会很多。这也实现了我们刚开始说的，前多少位必须为0的哈希值才算是满足要求。因此，也就需要通过一些困难的工作（一系列反复地计算），才能找到一个有效的数字。

现在，我们需要一些数据来进行哈希，准备数据：

~~~~go
func (pow *ProofOfWork) prepareData(nonce int) []byte {
    data := bytes.Join(
        [][]byte{
            pow.block.PrevBlockHash,
            pow.block.Data,
            IntToHex(pow.block.Timestamp),
            IntToHex(int64(targetBits)),
            IntToHex(int64(nonce)),
        },
        []byte{},
    )

    return data
}
~~~~

这个部分比较直观：只需要将 target ，nonce 与 Block 进行合并。这里的 nonce，就是上面 Hashcash 所提到的计数器，它是一个密码学术语。

其中有一个函数原文中并没有提到，那就是：IntToHex。

它实现了由int向其十六进制[]byte的转换。而这个函数在go的库中并没有实现。我们新建文件utils.go，并实现该函数：

~~~~~~go
func IntToHex(num int64) []byte{
    buff := new(bytes.Buffer)
    err := binary.Write(buff, binary.BigEndian, num)
    if err != nil {
        log.Panic(err)
    }

    return buff.Bytes()
}
~~~~~~

好，所有的准备工作已经完成了，下面来实现 PoW 算法的核心：

~~~~go
// Run performs a proof-of-work
func (pow *ProofOfWork) Run() (int, []byte) {
    var hash [32]byte
    var hashInt big.Int
    nonce := 0

    // fmt.Println("Mining the block containing \" %s \"", pow.block.Data)
    fmt.Printf("Mining the block containing \"%s\"\n", pow.block.Data)
    for nonce < maxNonce{
        data := pow.PrepareData(nonce)
        hash = sha256.Sum256(data)
        // fmt.Printf("\r%x", hash)
        hashInt.SetBytes(hash[:])

        if hashInt.Cmp(pow.target) == -1 {
            break
        } else {
            nonce++
        }
    }
    fmt.Printf("\r%x\n\n", hash)

    return nonce, hash[:]
}
~~~~

这段代码与原教程中有所不同，主要在于输出部分。原教程由于在挖矿过程中输出了每一个哈希值而加入了过多的输出，导致程序运行效率太过低下。

首先我们对变量进行初始化：

* HashInt 是 hash 的整形表示；
* nonce 是计数器。

然后开始一个 “无限” 循环,maxNonce 对这个循环进行了限制, 它等于 math.MaxInt64，这是为了避免 nonce 可能出现的溢出。尽管我们 pow的难度很小，以至于计数器其实不太可能会溢出，但最好还是以防万一检查一下。
在这个循环中，我们做的事情有：

* 准备数据
* 用 SHA-256 对数据进行哈希
* 将哈希转换成一个大整数
* 将这个大整数与目标进行比较

跟之前所讲的一样简单。现在我们可以移除 Block 的 SetHash 方法，然后修改 NewBlock 函数：

~~~~go
func NewBlock(data string, prevBlockHash []byte) *Block {
    block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}, 0}
    pow := NewProofOfWork(block)
    nonce, hash := pow.Run()

    block.Hash = hash[:]
    block.Nonce = nonce

    return block
}
~~~~~~~~

在这里，你可以看到 nonce 被保存为 Block 的一个属性。这是十分有必要的，因为待会儿我们对这个工作量进行验证时会用到 nonce 。Block 结构现在看起来像是这样：

~~~~go
type Block struct {
    Timestamp     int64
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
    Nonce         int
}
~~~~~

好了！现在让我们来运行一下是否正常工作：这一过程可能会十分的缓慢，尤其是我们挖矿过程中每次尝试的哈希值都会输出，这极大地增加了程序运行时间。这里建议按照我的思路将proofOfWork.go中run()函数里面的"fmt.Printf("\r%x", hash)"
注释掉。）

![smiley](\assets\images\usedInBlogs\blockChainGo\2\3.png)

好的，你可以看到我们找到的每一个哈希值都是以六个0开始的。这个过程需要很多的时间。

至此，我们还有一个事情没有做，那就是完成“工作量的证明”的证实过程。


~~~~go
func (pow *ProofOfWork) Validate() bool {
    var hashInt big.Int

    data := pow.prepareData(pow.block.Nonce)
    hash := sha256.Sum256(data)
    hashInt.SetBytes(hash[:])

    isValid := hashInt.Cmp(pow.target) == -1

    return isValid
}
~~~~

这也正是我们为什么要存储Nonce

我们再次检查是否一切运行正确：

~~~~go
func main() {
    bc := NewBlockchain()

    bc.AddBlock("Send 1 BTC to Ivan")
    bc.AddBlock("Send 2 more BTC to Ivan")

    for _, block := range bc.Blocks{
        fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
        fmt.Printf("Data: %s\n", block.Data)
        fmt.Printf("Hash: %x\n", block.Hash)
        pow := NewProofOfWork(block)
        fmt.Printf("PoW: %s\n", strconv.FormatBool(pow.Validate())) //验证工作证明
        fmt.Println()
    }
}
~~~~

![smiley](\assets\images\usedInBlogs\blockChainGo\2\4.png)

可以看到，两次运行得到的有效哈希值并非完全相同。

我们的块链是一个更接近其实际架构的一步：添加块现在需要工作证明，因此mining是可能的。但是它仍然缺乏一些关键的特征：块链数据库不是持久性数据，没有钱包，地址，交易，没有共识机制。所有这些我们将在以后的文章中实现。祝您挖矿愉快！


未解释：

接口，函数格式
PrepareDat ,

// fmt.Println("Mining the block containing \" %s \"", pow.block.Data)

        fmt.Printf("\r%x", hash)

        printf, print, println

        NewProofOfWork中为什么结果不同