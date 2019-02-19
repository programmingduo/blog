---
layout: post
title: "block chain go (3)：Persistence and CLI"
description: "区块链和比特币学习博客"
categories: [block-chains]
tags: [block-chains,golang]
redirect_from:
  - /2018/09/27/
---

* Karmdown table of content
{:toc .toc}

# 原文链接
[Building Blockchain in Go. Part 3: Persistence and CLI](https://jeiwan.cc/posts/building-blockchain-in-go-part-3/)

# 简介
目前我们构建的区块链已经实现了工作量证明的过程，这也使得挖矿得以实施。但是区块链的本质是一个分布式数据库，而目前为止我们所做的所有工作并不能存储数据或者读取数据。而这一节的内容将会实现在数据库中存储区块链的工作，并且会简单实现一个命令行界面来执行一些区块链的操作。

# 数据库选择
目前为止，我们实现的区块链中还没有数据库。我们每次创建区块的时候都将他们存储在内存中。那么一旦程序退出，所有的内容就都消失了。我们没有办法再次使用这条链，也没有办法与其他人共享，所以我们需要把它存储到磁盘上。

那我们选择哪种数据库呢？事实上，任何一种数据库都可以满足我们的需求。在[比特币原始论文](https://bitcoin.org/bitcoin.pdf)中，并没有提到要使用哪一个具体的数据库，它完全取决于开发者如何选择。[Bitcoin Core](https://github.com/bitcoin/bitcoin) ，最初由中本聪发布，现在是比特币的一个参考实现，它使用的是 [LevelDB](https://github.com/google/leveldb)。而我们将要使用的是另一种数据库：

# BoltDB

因为它：

* 非常简洁
* 用 Go 实现
* 不需要运行一个服务器
* 能够允许我们构造想要的数据结构

BoltDB GitHub 上的 [README](https://github.com/boltdb/bolt) 是这么说的：
Bolt 是一个纯键值存储的 Go 数据库，启发自 Howard Chu 的 LMDB. 它旨在为那些无须一个像Postgres和MySQL这样有着完整数据库服务器的项目，提供一个简单，快速和可靠的数据库。
由于 Bolt 意在用于提供一些底层功能，简洁便成为其关键所在。它的 API 并不多，并且仅关注值的获取和设置。仅此而已。

听起来非常符合我们的要求(坦白讲我第一次读原文的时候并没有发现有啥符合的地方。如果你跟我一样，建议你这篇文章读两次。。第二次就能get到点了)。我们来回顾、阐述一下这些特征。

BoltDB是一种键值对存储的数据库，这意味着它不并不像SQL RDBMS (MySQL, PostgreSQL, 等等)一样有表，也没有行和列。它很像是Golang中的maps。键值对被存储在buckets（你可以想象成一个桶或者容器）里面，这个buckets的存在是为了将同类型的数据（键值对）放在一起。因此，为了获得数据库中的一个值，你需要知道bucket和键。

关于BoltDB还有一点非常重要。BoltDB存储的数据当中并不区分数据的类型，而是统一存储数据的比特数组。而为了存储我们的block，我们需要将block的数据进行序列化，也就说，实现一个从 Go struct 转换到一个 byte array 的机制，同时还可以从一个 byte array 再转换回 Go struct。虽然我们将会使用 encoding/gob 来完成这一目标，但实际上也可以选择使用 JSON, XML, Protocol Buffers 等等。之所以选择使用 encoding/gob, 是因为它很简单，而且是 Go 标准库的一部分。

虽然 BoltDB 的作者出于个人原因已经不在对其维护（见[README](https://github.com/boltdb/bolt/commit/fa5367d20c994db73282594be0146ab221657943)）, 不过关系不大，它已经足够稳定了，况且也有活跃的 [fork](https://github.com/etcd-io/bbolt)

# 数据库结构

在开始实现持久化的逻辑之前，我们首先需要决定到底要如何在数据库中进行存储。为此，我们可以参考 Bitcoin Core 的做法：

简单来说，Bitcoin Core 使用两个 “bucket” 来存储数据：
* 其中一个 bucket 是 blocks，它存储了描述一条链中所有块的元数据
* 另一个 bucket 是 chainstate，存储了一条链的状态，也就是当前所有的未花费的交易输出，和一些元数据

此外，出于性能的考虑，Bitcoin Core 将每个区块（block）存储为磁盘上的不同文件。如此一来，就不需要仅仅为了读取一个单一的块而将所有（或者部分）的块都加载到内存中。但是，为了简单起见，我们并不会实现这一点。

在 blocks 中，key -> value（键值对）(暂时还用不着吧)为：

* key	value
* b + 32 字节的 block hash	block index record
* f + 4 字节的 file number	file information record
* l + 4 字节的 file number	the last block file number used
* R + 1 字节的 boolean	是否正在 reindex
* F + 1 字节的 flag name length + flag name string	1 byte boolean: various flags that can be on or off
* t + 32 字节的 transaction hash	transaction index record


在 chainstate，key -> value 为：

* key	value
* c + 32 字节的 transaction hash	unspent transaction output record for that transaction
* B	32 字节的 block hash: the block hash up to which the database represents the unspent transaction outputs

因为目前还没有交易，所以我们只需要 blocks bucket。另外，正如上面提到的，我们会将整个数据库存储为单个文件，而不是将区块存储在不同的文件中。所以，我们也不会需要文件编号（file number）相关的东西。最终，我们会用到的键值对有：

* 32 字节的 block-hash -> block 结构
* l -> 链中最后一个块的 hash

这就是实现持久化机制所有需要了解的内容了。

# 序列化

上面提到，在 BoltDB 中，值只能是 []byte 类型，但是我们想要存储 Block 结构。所以，我们需要使用 encoding/gob 来对这些结构进行序列化。

让我们来实现 Block 的 Serialize 方法（为了简洁起见，此处略去了错误处理）：

~~~go
func (b *Block) Serialize() []byte {
	var result bytes.Buffer
	encoder := gob.NewEncoder(&result)

	err := encoder.Encode(b)
	if err != nil{
		log.Panic(err)
	}

	return result.Bytes()
}
~~~~

如果有需要可自行查阅[encoding包](http://docscn.studygolang.com/pkg/encoding/gob/)

另外，原教程中的代码上述代码并非完全一致，主要在于博主自行加入了err是否为空的判断。如果不加则会产生报错。并且对于err的检测从逻辑上也应该是代码中的一部分。

缘教程的代码这里也贴一下。这一节中的缘教程并没有任何err检测。下面部分的err检测也都是博主自行加入的。

Serialize()源代码

~~~go
func (b *Block) Serialize() []byte {
	var result bytes.Buffer
	encoder := gob.NewEncoder(&result)

	err := encoder.Encode(b)

	return result.Bytes()
}
~~~~~

这个部分比较直观：首先，我们定义一个 buffer 存储序列化之后的数据。然后，我们初始化一个 gob encoder 并对 block 进行编码，结果作为一个字节数组返回。

接下来，我们需要一个解序列化的函数，它会接受一个字节数组作为输入，并返回一个 Block. 它不是一个方法（method），而是一个单独的函数（function）[tips]：

~~~go
func DeserializeBlock(d []byte) *Block {
	var block Block

	decoder := gob.NewDecoder(bytes.NewReader(d))
	err := decoder.Decode(&block)
	if err != nil{
		log.Panic(err)
	}

	return &block
}
~~~~

这就是序列化部分的内容了。

# 持久化

让我们从 NewBlockchain 函数开始。在之前的实现中，NewBlockchain 会创建一个新的 Blockchain 实例，并向其中加入创世块。而现在，我们希望它做的事情有：

* 打开一个数据库文件
* 检查文件里面是否已经存储了一个区块链
* 如果已经存储了一个区块链：
	* 创建一个新的 Blockchain 实例
	* 设置 Blockchain 实例的 tip 为数据库中存储的最后一个块的哈希
* 如果没有区块链：
	* 创建创世块
	* 存储到数据库
	* 将创世块哈希保存为最后一个块的哈希
	* 创建一个新的 Blockchain 实例，初始时 tip 指向创世块（tip 有尾部，尖端的意思，在这里 tip 存储的是最后一个块的哈希）

代码大概是这样：

~~~go
func NewBlockchain() *Blockchain{
	var tip []byte
	db, err := bolt.Open(dbFile, 0600, nil)
	if err != nil{
		log.Panic(err)
	}

	err = db.Update(func(tx *bolt.Tx) error{
			b := tx.Bucket([]byte(blocksBucket))

			if b == nil{
				genesis := NewGenesisBlock()
				b, err := tx.CreateBucket([]byte(blocksBucket))
				err = b.Put(genesis.Hash, genesis.Serialize())
				err = b.Put([]byte("1"), genesis.Hash)
				if err != nil{
					log.Panic(err)
				}
				tip = genesis.Hash
			}else {
				tip = b.Get([]byte("1"))
			}

			return nil
		})
	if err != nil{
		log.Panic(err)
	}

	bc := Blockchain{tip, db}

	return &bc

}
~~~~

博主这里建议先阅读一下 [boltdb 学习和实践](https://segmentfault.com/a/1190000010098668 "boltdb 学习和实践")

boltdb在上一链接已经阐述的很详尽了，这里只赘述其中一点，在import  "github.com/boltdb/bolt" 之前，首先要在cmd中安装boltdb。安装命令：

> go get github.com/boltdb/bolt

之后我们接着看代码。让我们来一段一段的解释一下上述代码。

~~~go
db, err := bolt.Open(dbFile, 0600, nil)
~~~~~

这是打开一个BoltDB文件的标准做法。注意，即使不存在这样的文件，它也不会返回错误，而会建立这样一个文件。

"dbFile" 是一个常量。原教程中并未说明。但是我们应当在blockChain全局中加入这样一行代码：

~~~go
const dbFile = "blockchain.db"
~~~~~

~~~~go
err = db.Update(func(tx *bolt.Tx) error {
...
})
~~~~

在 BoltDB 中，数据库操作通过一个事务（transaction）进行操作。有两种类型的事务：只读（read-only）和读写（read-write）。这里，打开的是一个读写事务（db.Update(...)），因为我们可能会向数据库中添加创世块。

~~~~go
b := tx.Bucket([]byte(blocksBucket))

if b == nil{
	genesis := NewGenesisBlock()
	b, err := tx.CreateBucket([]byte(blocksBucket))
	err = b.Put(genesis.Hash, genesis.Serialize())
	err = b.Put([]byte("1"), genesis.Hash)
	if err != nil{
		log.Panic(err)
	}
	tip = genesis.Hash
}else {
	tip = b.Get([]byte("1"))
}
~~~~~~

代码中的"bucket" 也是一个常量。同样要在全局中加入语句

~~~go
const blocksBucket = "blocks"  
~~~~~

这里是函数的核心。在这里，我们先获取了存储区块的 bucket：如果存在，就从中读取 l 键；如果不存在，就生成创世块，创建 bucket，并将区块保存到里面，然后更新 l 键以存储链中最后一个块的哈希。

另外，注意创建 Blockchain 一个新的方式：

~~~go
bc := Blockchain{tip, db}
~~~~

这次，我们不在里面存储所有的区块了，而是仅存储区块链的 tip。另外，我们存储了一个数据库连接。因为我们想要一旦打开它的话，就让它一直运行，直到程序运行结束。因此，Blockchain 的结构现在看起来是这样：

~~~go
type Blockchain struct{
	tip []byte
	db *bolt.DB
}
~~~~~

接下来我们想要更新的是 AddBlock 方法：现在向链中加入区块，就不是像之前向一个数组中加入一个元素那么简单了。从现在开始，我们会将区块存储在数据库里面：

~~~~go
func (bc *Blockchain) AddBlock(data string) {
	var lastHash []byte 
	err := bc.db.View(func (tx *bolt.Tx) error{
			b := tx.Bucket([]byte(blocksBucket))
			lastHash = b.Get([]byte("1"))
			 return nil
		})
	if err != nil{
		log.Panic(err)
	}

	newBlock := NewBlock(data, lastHash)

	err = bc.db.Update(func (tx *bolt.Tx) error{
			b := tx.Bucket([]byte(blocksBucket))
			err := b.Put(newBlock.Hash, newBlock.Serialize())
			if err != nil{
				log.Panic(err)
			}
			err = b.Put([]byte("1"), newBlock.Hash)
			if err != nil{
				log.Panic(err)
			}
			bc.tip = newBlock.Hash

			return nil
		})
	if err != nil{
		log.Panic(err)
	}
}
~~~~~~

只说明一点，我们将最后一个存入的区块映射到字符串"1"中，然后读取的时候也只是读取最后一个区块。

# 检查区块链

现在，产生的所有块都会被保存到一个数据库里面，所以我们可以重新打开一个链，然后向里面加入新块。但是在实现这一点后，我们失去了之前一个非常好的特性：再也无法打印区块链的区块了，因为现在不是将区块存储在一个数组，而是放到了数据库里面。让我们来解决这个问题！

BoltDB 允许对一个 bucket 里面的所有 key 进行迭代，但是所有的 key 都以字节序进行存储，而且我们想要以区块能够进入区块链中的顺序进行打印。此外，因为我们不想将所有的块都加载到内存中（因为我们的区块链数据库可能很大！或者现在可以假装它可能很大），我们将会一个一个地读取它们。故而，我们需要一个区块链迭代器（BlockchainIterator）：

~~~go
type BlockchainIterator struct {
	currentHash []byte
	db *bolt.DB
}
~~~~~

每当要对链中的块进行迭代时，我们就会创建一个迭代器，里面存储了当前迭代的块哈希（currentHash）和数据库的连接（db）。通过 db，迭代器逻辑上被附属到一个区块链上（这里的区块链指的是存储了一个数据库连接的 Blockchain 实例），并且通过 Blockchain 方法进行创建：

~~~go
func (bc *Blockchain) Iterator() *BlockchainIterator{
	bci := &BlockchainIterator{bc.tip, bc.db}
	return bci
}
~~~~

注意，迭代器的初始状态为链中的 tip，因此区块将从尾到头（创世块为头），也就是从最新的到最旧的进行获取。实际上，选择一个 tip 就是意味着给一条链“投票”。一条链可能有多个分支，最长的那条链会被认为是主分支。在获得一个 tip （可以是链中的任意一个块）之后，我们就可以重新构造整条链，找到它的长度和需要构建它的工作。这同样也意味着，一个 tip 也就是区块链的一种标识符。

BlockchainIterator 只会做一件事情：返回链中的下一个块。

~~~~go
func (i *BlockchainIterator) Next() *Block{
	var block *Block
	err := i.db.View(func(tx *bolt.Tx) error{
			b := tx.Bucket([]byte(blocksBucket))
			encoder := b.Get(i.currentHash)
			block = DeserializeBlock(encoder)

			return nil
		})
	if err != nil{
		log.Panic(err)
	}

	i.currentHash = block.PrevBlockHash
	return block
}
~~~~~

这就是数据库部分的内容了！

# CLI

到目前为止，我们的实现还没有提供一个与程序交互的接口：目前只是在 main 函数中简单执行了 NewBlockchain 和 bc.AddBlock 。是时候改变了！现在我们想要拥有这些命令：

> blockchain_go addblock "Pay 0.031337 for a coffee"
> blockchain_go printchain

所有命令行相关的操作都会通过 CLI 结构进行处理：

~~~go
type CLI struct{
	bc *Blockchain
}
~~~~

它的 “入口” 是 Run 函数：

~~~go
func (cli *CLI) Run(){
	// cli.validateArgs()

	addBlockCmd := flag.NewFlagSet("addblock", flag.ExitOnError)
	printChainCmd := flag.NewFlagSet("printchain", flag.ExitOnError)

	addBlockData := addBlockCmd.String ("data", "", "Block data")
	
	switch os.Args[1] {
	case "addblock":
		err := addBlockCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	case "printchain":
		err := printChainCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	default :
		// cli.printUsage()
		os.Exit(1)
	}

	if addBlockCmd.Parsed(){
		if *addBlockData == ""{
			addBlockCmd.Usage()
			os.Exit(1)
		}
		cli.addBlock(*addBlockData)
	}

	if printChainCmd.Parsed(){
		cli.printChain()
	}
}
~~~~~

让我们来看一下这段代码。

我们使用标准库中的 [flag](http://docscn.studygolang.com/pkg/flag/)包来解析命令行参数。

这里拿出"addblock"命令实现的代码进行分析。

~~~~go
	addBlockCmd := flag.NewFlagSet("addblock", flag.ExitOnError)

	addBlockData := addBlockCmd.String ("data", "", "Block data")

	switch os.Args[1] {
		case "addblock":
			err := addBlockCmd.Parse(os.Args[2:])
			if err != nil{
				log.Panic(err)
			}
			...
	}

	if addBlockCmd.Parsed(){
		if *addBlockData == ""{
			addBlockCmd.Usage()
			os.Exit(1)
		}
		cli.addBlock(*addBlockData)
	}

~~~~~~~

这段代码实现了如下功能：
	加入了命令"addblock"
	为命令"addblock"加入了参数"data"，类型为string
	解析参数"data"的内容
	如果解析了"data"，则调用函数cli.addBlock

printChainCmd的代码逻辑类似，不做解释。

接着检查解析是哪一个子命令，并调用相关函数：

~~~go
func (cli *CLI) addBlock(data string){
	cli.bc.AddBlock(data)
	fmt.Println("Success!")
}

func (cli *CLI) printChain(){
	bci := cli.bc.Iterator()

	for{
		block := bci.Next()

		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		pow := NewProofOfWork(block)
		fmt.Printf("PoW: %s\n", strconv.FormatBool(pow.Validate()))
		fmt.Println()

		if len(block.PrevBlockHash) == 0 {
			break
		}
	}
}
~~~~~

这部分内容跟之前的很像，唯一的区别是我们现在使用的是 BlockchainIterator 对区块链中的区块进行迭代：
记得不要忘了对 main 函数作出相应的修改：

~~~go
func main() {
    bc := NewBlockchain()
    defer bc.db.Close()

    cli := CLI{bc}
    cli.Run()
}
~~~~~

注意，无论提供什么命令行参数，都会创建一个新的链。
这就是今天的所有内容了! 来看一下是不是如期工作：

![smiley](\assets\images\usedInBlogs\blockChainGo\3\1.png)













未解释部分：

method function

gob

gob:  http://docscn.studygolang.com/pkg/encoding/gob/

https://segmentfault.com/a/1190000010098668

go get github.com/boltdb/bolt

defer  https://blog.csdn.net/skh2015java/article/details/77081250