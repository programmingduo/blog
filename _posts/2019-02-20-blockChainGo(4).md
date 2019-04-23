---
layout: post
title: "Building Blockchain in Go. Part 4: Transactions 1"
description: "区块链和比特币学习博客"
categories: [block-chains]
tags: [block-chains,golang]
redirect_from:
  - /2019/02/20/
---

* Karmdown table of content
{:toc .toc}

# 原文链接
[Building Blockchain in Go. Part 4: Transactions 1](https://jeiwan.cc/posts/building-blockchain-in-go-part-4/)

# 引言

交易（transaction）是比特币的核心所在，而区块链唯一的目的，也正是为了能够安全可靠地存储交易。在区块链中，交易一旦被创建，就没有任何人能够再去修改或是删除它。今天，我们将会开始实现交易。不过，由于交易是很大的话题，我会把它分为两部分来讲：在今天这个部分，我们会实现交易的基本框架。在第二部分，我们会继续讨论它的一些细节。
由于比特币采用的是[UTXO 模型](https://new.qq.com/omn/20180309/20180309G10YO0.html)，并非账户模型，并不直接存在“余额”这个概念，余额需要通过遍历整个交易历史得来。

# There is no spoon
此标题无法翻译，建议参考[百度](https://baike.baidu.com/item/There%20is%20no%20spoon/4913256?fr=aladdin)

如果你曾经开发过Web应用程序，为了实现付款，可能会在数据库中创建这些表：帐户和事务。一个账户将存储关于一个用户的信息，包括他们的个人信息和余额，而一个交易将存储关于从一个账户到另一个账户转账的信息。比特币的支付方式完全不同：

* 没有帐号。
* 没有余额。
* 没有地址。
* 没有硬币。
* 没有发送者和接收者。

由于区块链是一个公开的数据库，我们不想存储钱包所有者的敏感信息。钱币不计入账户。交易不会将钱从一个地址转移到另一个地址。没有保存帐户余额的字段或属性。只有交易记录。但在交易中有什么？

# 比特币交易

点击 [这里](https://www.blockchain.com/zh-cn/btc/tx/b6f6b339b546a13822192b06ccbdd817afea5311845f769702ae2912f7d94ab5) 在 blockchain.info 查看下图中的交易信息。

![smiley](\assets\images\usedInBlogs\blockChainGo\4\1.png)

一笔交易由一些输入（input）和输出（output）组合而来。新建文件transaction.go: 

~~~go
type Transaction struct{
	ID []byte
	Vin []TXInput
	Vout []TXOutput
}
~~~~

对于每一笔新的交易，它的输入会引用之前一笔交易的输出（这里有个例外，coinbase 交易，也就是矿工挖矿过程中的比特币交易），引用就是花费的意思。所谓引用之前的一个输出，也就是将之前的一个输出包含在另一笔交易的输入当中，就是花费之前的交易输出。交易的输出，就是币实际存储的地方。如果这一部分理解不了建议查阅NTXO模型相关资料。下面的图示阐释了交易之间的互相关联：

![smiley](\assets\images\usedInBlogs\blockChainGo\4\2.png)

注意：

* 1.有一些输出并没有被关联到某个输入上
* 2.一笔交易的输入可以引用之前多笔交易的输出
* 3.一个输入必须引用至少一个输出
* 4.输入的总和一定要大于或者输出。一部分用于矿工的酬劳，其余部分找零

贯穿本文，我们将会使用像“钱（money）”，“币（coin）”，“花费（spend）”，“发送（send）”，“账户（account）” 等等这样的词。但是在比特币中，其实并不存在这样的概念。交易仅仅是通过一个脚本（script）来锁定（lock）一些值（value），而这些值只可以被锁定它们的人解锁（unlock）。
每一笔比特币交易都会创造输出，输出都会被区块链记录下来。给某个人发送比特币，实际上意味着创造新的 UTXO 并注册到那个人的地址，可以为他所用。

# 交易输出

先从输出（output）开始：

~~~go
type TXOutput struct{
	Value int
    ScriptPubKey string
}
~~~~

输出主要包含两部分：

* 一定量的比特币(Value)
* 一个锁定脚本(ScriptPubKey)，要花这笔钱，必须要解锁该脚本。

实际上，正是输出里面存储了“币”（注意，也就是上面的 Value 字段）。而这里的存储，指的是用一个数学难题对输出进行锁定，这个难题被存储在 ScriptPubKey 里面。在内部，比特币使用了一个叫做 Script 的脚本语言，用它来定义锁定和解锁输出的逻辑。虽然这个语言相当的原始（这是为了避免潜在的黑客攻击和滥用而有意为之），并不复杂，但是我们也并不会在这里讨论它的细节。你可以在[这里](https://en.bitcoin.it/wiki/Script) 找到详细解释。

在比特币中，value 字段存储的是 satoshi 的数量，而不是 BTC 的数量。一个 satoshi 等于一亿分之一的 BTC(0.00000001 BTC)，这也是比特币里面最小的货币单位（就像是 1 分的硬币）。

由于还没有实现地址（address），所以目前我们会避免涉及逻辑相关的完整脚本。ScriptPubKey 将会存储一个任意的字符串（用户定义的钱包地址）。

顺便说一下，有了一个这样的脚本语言，也意味着比特币其实也可以作为一个智能[合约平台](https://baike.baidu.com/item/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6/19770937?fr=aladdin)。

关于输出，非常重要的一点是：它们是不可再分的（indivisible）。也就是说，你无法仅引用它的其中某一部分。要么不用，如果要用，必须一次性用完。当一个新的交易中引用了某个输出，那么这个输出必须被全部花费。如果它的值比需要的值大，那么就会产生一个找零，找零会返还给发送方。这跟现实世界的场景十分类似，当你想要支付的时候，如果一个东西值 1 美元，而你给了一个 5 美元的纸币，那么你会得到一个 4 美元的找零。

# 交易输入

这里是输入：

~~~go
type TXInput struct{
	Txid []byte
	//存储的是之前交易的 ID
    Vout int
    //存储的是该输出在那笔交易中所有输出的索引
    ScriptSig string
}
~~~~~

正如之前所提到的，一个输入引用了之前交易的一个输出：Txid 存储的是之前交易的 ID，Vout 存储的是该输出在那笔交易中所有输出的索引（因为一笔交易可能有多个输出，需要有信息指明是具体的哪一个）。ScriptSig 是一个脚本，提供了可解锁输出结构里面 ScriptPubKey 字段的数据。如果 ScriptSig 提供的数据是正确的，那么输出就会被解锁，然后被解锁的值就可以被用于产生新的输出；如果数据不正确，输出就无法被引用在输入中，或者说，无法使用这个输出。这种机制，保证了用户无法花费属于其他人的币。

再次强调，由于我们还没有实现地址，所以目前 ScriptSig 将仅仅存储一个用户自定义的任意钱包地址。我们会在下一篇文章中实现公钥（public key）和签名（signature）。
来简要总结一下。输出，就是 “币” 存储的地方。每个输出都会带有一个解锁脚本，这个脚本定义了解锁该输出的逻辑。每笔新的交易，必须至少有一个输入和输出。一个输入引用了之前一笔交易的输出，并提供了解锁数据（也就是 ScriptSig 字段），该数据会被用在输出的解锁脚本中解锁输出，解锁完成后即可使用它的值去产生新的输出。
每一笔输入都是之前一笔交易的输出，那么假设从某一笔交易开始不断往前追溯，它所涉及的输入和输出到底是谁先存在呢？换个说法，这是个鸡和蛋谁先谁后的问题，是先有蛋还是先有鸡呢？

# 先有蛋

在比特币中，是先有蛋，然后才有鸡。输入引用输出的逻辑，是经典的“蛋还是鸡”问题：输入先产生输出，然后输出使得输入成为可能。在比特币中，最先有输出，然后才有输入。换而言之，第一笔交易只有输出，没有输入。
当矿工挖出一个新的块时，它会向新的块中添加一个 coinbase 交易。coinbase 交易是一种特殊的交易，它不需要引用之前一笔交易的输出。它“凭空”产生了币（也就是产生了新币），这是矿工获得挖出新块的奖励，也可以理解为“发行新币”。
在区块链的最初，也就是第一个块，叫做创世块。正是这个创世块，产生了区块链最开始的输出。对于创世块，不需要引用之前的交易输出。因为在创世块之前根本不存在交易，也就没有不存在交易输出。
来创建一个 coinbase 交易：

~~~go
const subsidy = 10

func NewCoinbaseTX(to string, data string) *Transaction{
	if data == ""{
		data = fmt.Sprintf("Reward to '%s'", to)
	}

	txin := TXInput{[]byte{}, -1, data}
    txout := TXOutput{subsidy, to}
    tx := Transaction{nil, []TXInput{txin}, []TXOutput{txout}}
    tx.SetID()

    return &tx
}

// SetID sets ID of a transaction
func (tx *Transaction) SetID() {
	var encoded bytes.Buffer
	var hash [32]byte

	enc := gob.NewEncoder(&encoded)
	err := enc.Encode(tx)
	if err != nil {
		log.Panic(err)
	}
	hash = sha256.Sum256(encoded.Bytes())
	tx.ID = hash[:]
}

~~~~

coinbase 交易只有一个输出，没有输入。在我们的实现中，它表现为 Txid 为空，Vout 等于 -1。并且，在当前实现中，coinbase 交易也没有在 ScriptSig 中存储脚本，而只是存储了一个任意的字符串 data。


在比特币中，第一笔 coinbase 交易包含了如下信息：“The Times 03/Jan/2009 Chancellor on brink of second bailout for banks”。[可点击这里查看](https://www.blockchain.com/btc/tx/4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b?show_adv=true)。

subsidy 是挖出新块的奖励金。在比特币中，实际并没有存储这个数字，而是基于区块总数进行计算而得：区块总数除以 210000 就是 subsidy。挖出创世块的奖励是 50 BTC，每挖出 210000 个块后，奖励减半。在我们的实现中，这个奖励值将会是一个常量（至少目前是）。

# 将交易保存到区块链

从现在开始，每个块必须存储至少一笔交易。如果没有交易，也就不可能出新的块。这意味着我们应该移除 Block 的 Data 字段，取而代之的是存储交易：

~~~go
type Block struct{
	TimeStamp int64
	Transactions []*Transaction //
	PrevBlockHash []byte
	Hash []byte
	Nonce int
}
~~~~

NewBlock 和 NewGenesisBlock 也必须做出相应改变：

~~~go
func NewBlock(transactions []*Transaction, prevBlockHash []byte) *Block {
	block := &Block{time.Now().Unix(), transactions, prevBlockHash,  []byte{}, 0} //
	pow := NewProofOfWork(block)
	nonce, hash := pow.Run()

	block.Hash = hash[:]
	block.Nonce = nonce

	return block
}

func NewGenesisBlock(coinbase *Transaction) *Block {
	return NewBlock([]*Transaction{coinbase}, []byte{}) //
}
~~~~~

接下来修改创建区块链的函数（之前只有NewBlockChain函数，但其实这个函数实现的功能更像是GetBlockChain，所以我把函数名改为了GetBlockChain。但是这里创建区块链的函数是另一个函数，命名为CreateBlockChain）：

~~~go
func CreateBlockchain(address string) *Blockchain{
	if dbExists(){
		fmt.Println("Blockchain already exit")
		os.Exit(1)
	}

	var tip []byte
	db,err := bolt.Open(dbFile, 0600, nil)
	if err != nil{
		log.Panic(err)
	}

	err = db.Update(func(tx *bolt.Tx) error{
		cbtx := NewCoinbaseTX(address, genesisCoinbaseData)
		genesis := NewGenesisBlock(cbtx)

		b, err := tx.CreateBucket([]byte(blocksBucket))
		if err != nil {
			log.Panic(err)
		}

		err = b.Put(genesis.Hash, genesis.Serialize())
		if err != nil {
			log.Panic(err)
		}

		err = b.Put([]byte("1"), genesis.Hash)
		if err != nil {
			log.Panic(err)
		}
		tip = genesis.Hash

		return nil
		})

	if err != nil {
		log.Panic(err)
	}

	bc := Blockchain{tip, db}

	return &bc
}

~~~~~~

现在，这个函数会接受一个地址作为参数，这个地址将会被用来接收挖出创世块的奖励。

# 工作量证明

工作量证明算法必须要将存储在区块里面的交易考虑进去，从而保证区块链交易存储的一致性和可靠性。所以，我们必须修改 ProofOfWork.prepareData 方法：

不像之前使用 pow.block.Data，现在我们使用 pow.block.HashTransactions() ：

~~~go
func (pow *ProofOfWork) PrepareData(nonce int) []byte{
	data := bytes.Join(
		[][]byte{
			pow.block.PrevBlockHash,
			pow.block.HashTransactions(),
			IntToHex(pow.block.TimeStamp),
			IntToHex(int64(targetBits)),
			IntToHex(int64(nonce)),
		},
		[]byte{},
		)

	return data
}
~~~~


~~~go
func (b *Block)HashTransactions() []byte{
	var txHashes [][]byte
    var txHash [32]byte

    for _, tx := range b.Transactions {
        txHashes = append(txHashes, tx.ID)
    }
    txHash = sha256.Sum256(bytes.Join(txHashes, []byte{}))

    return txHash[:]
}
~~~~

通过哈希提供数据的唯一表示，这种做法我们已经不是第一次遇到了。我们想要通过仅仅一个哈希，就可以识别一个块里面的所有交易。为此，先获得每笔交易的哈希，然后将它们关联起来，最后获得一个连接后的组合哈希。

比特币使用了一个更加复杂的技术：它将一个块里面包含的所有交易表示为一个 Merkle tree ，然后在工作量证明系统中使用树的根哈希（root hash）。这个方法能够让我们快速检索一个块里面是否包含了某笔交易，即只需 root hash 而无需下载所有交易即可完成判断。

原教程中已经可以检测是否可以创建创世块了。但是其实还不可以编译。所以这里没有效果图。

嘴上说着不可以编译，还是照着人家的代码加上自己的一通注释、调试，做出了该有的样子。

![smiley](\assets\images\usedInBlogs\blockChainGo\4\3.png)

接下来我们将会实现余额查询

# 未花费交易输出

我们需要找到所有的未花费交易输出（unspent transactions outputs, UTXO）。未花费（unspent） 指的是这个输出还没有被包含在任何交易的输入中，或者说没有被任何输入引用。在上面的图示中，未花费的输出是：

* tx0, output 1;
* tx1, output 0;
* tx3, output 0;
* tx4, output 0.

当然了，检查余额时，我们并不需要知道整个区块链上所有的 UTXO，只需要关注那些我们能够解锁的那些 UTXO（目前我们还没有实现密钥，所以我们将会使用用户定义的地址来代替）。首先，让我们定义在输入和输出上的锁定和解锁方法：

~~~go

func (out *TXOutput)CanBeUnlockedWith(unlockingData string) bool{
	return out.ScriptPubKey == unlockingData
}

func (in *TXInput)CanUnlockOutputWith(unlockingData string) bool{
	return in.ScriptSig == unlockingData
}
~~~~~

在这里，我们只是将 script 字段与 unlockingData 进行了比较。在后续文章我们基于私钥实现了地址以后，会对这部分进行改进。
下一步，找到包含未花费输出的交易，这一步其实相当困难：

~~~go
func (bc *Blockchain) FindUnspentTransactions(address string)[]Transaction {
	bci := bc.Iterator()
	var unspentTXs []Transaction
  	spentTXOs := make(map[string][]int)

	for{
		//遍历区块链中的所有block
		block := bci.Next()
		//每一个block有可能存储着若干transactions
		for _, tx := range block.Transactions{
			txID := hex.EncodeToString(tx.ID)
		Outputs:
			//每一个transaction又有很多vout
			for outIdx, out := range tx.Vout{
				//判断这一vout是否能够被address解密
				if out.CanBeUnlockedWith(address){
					//如果可以又要判断是否被花费
					if(spentTXOs[txID] != nil){
						for sTX := range spentTXOs[txID]{
							if sTX == outIdx{
								continue Outputs
							}
						}
					}

					unspentTXs = append(unspentTXs, *tx)
				}
			}

			//判断是否被花费的过程中又需要存储所有已经花费的transaction
			if tx.IsCoinbase() == false {
				for _, in := range tx.Vin{
					if in.CanUnlockOutputWith(address) {
            			inTxID := hex.EncodeToString(in.Txid)
            			spentTXOs[inTxID] = append(spentTXOs[inTxID], in.Vout)
        		  	}
				}
			}

		}
		if len(block.PrevBlockHash) == 0 {
     		break
   		}
	}

	return unspentTXs
}
~~~~

这个过程却是相当繁杂。代码的逻辑实际上也可以看成事很多个for循环的一步步遍历以及筛选。

我们要想找到某address所有未花费的交易，需要遍历区块链中的所有block，然后每一个block有可能存储着若干transactions，每一个transaction又有很多vout。我们需要遍历所有的vout，判断这一vout是否能够被address解密。如果可以又要判断是否被花费。判断是否被花费的过程中又需要存储所有已经花费的transaction。

这里的实现方法与原教程不太一致。主要是考虑到检测某笔vout能否被address解密的系统开销要比判断其是否已经被支出要少一点。所以检测顺序有所不同。如果后续过程中判断解密过程更加复杂则可以视情况改变判断次序。

这个函数返回了一个交易列表，里面包含了未花费输出。为了计算余额，我们还需要一个函数将这些交易作为输入，然后仅返回一个输出。这个输出只返回某一个用户的为花费收入，是一个TXOutput的数组。而上一个函数返回的是该用户所有可花费的transaction：

~~~~go
func (bc *Blockchain) FindUTXO(address string) []TXOutput {
	uTX := bc.FindUnspentTransactions(address)
	var UTXO []TXOutput

	for _, tx := range uTX{
		for _, out := range tx.Vout{
			if out.CanBeUnlockedWith(address){
				UTXO = append(UTXO, out)
			}
		}
	}
	return UTXO
}

~~~~~

就是这么多了！现在我们来实现 getbalance 命令：

~~~~go
func (cli *CLI) getBalance(address string){
	bc := GetBlockchain()
	defer bc.db.Close()

	utxos := bc.FindUTXO(address)
	balance := 0
	for _, vout := range utxos{
		balance += vout.Value
	}

	fmt.Println("Balance of '%s' is %d", address, balance)
}
~~~~~

每一个章节当中都要更改cli文件。这里直接附上本章节所有代码完成之后的cli文件。如果读者逐步调试可以注释掉一些未定义的函数。

~~~go
package main

import(
	"fmt"
	"flag"
	"strconv"
	"os"
	"log"
)

type CLI struct{
	bc *Blockchain
}


func (cli *CLI) printUsage() {
	fmt.Println("Usage:")
	fmt.Println("  createblockchain -address ADDRESS - Create a blockchain and send genesis block reward to ADDRESS")
	fmt.Println("  createwallet - Generates a new key-pair and saves it into the wallet file")
	fmt.Println("  getbalance -address ADDRESS - Get balance of ADDRESS")
	fmt.Println("  listaddresses - Lists all addresses from the wallet file")
	fmt.Println("  printchain - Print all the blocks of the blockchain")
	fmt.Println("  reindexutxo - Rebuilds the UTXO set")
	fmt.Println("  send -from FROM -to TO -amount AMOUNT -mine - Send AMOUNT of coins from FROM address to TO. Mine on the same node, when -mine is set.")
	fmt.Println("  startnode -miner ADDRESS - Start a node with ID specified in NODE_ID env. var. -miner enables mining")
}

func (cli *CLI) addBlock(data string){
	// cli.bc.AddBlock(data)
	fmt.Println("Success!")
}

func (cli *CLI) printChain(){
	bci := cli.bc.Iterator()

	for{
		block := bci.Next()

		fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
		// fmt.Printf("Data: %s\n", block.Data)
		fmt.Printf("Hash: %x\n", block.Hash)
		pow := NewProofOfWork(block)
		fmt.Printf("PoW: %s\n", strconv.FormatBool(pow.Validate()))
		fmt.Println()

		if len(block.PrevBlockHash) == 0 {
			break
		}
	}
}

func (cli *CLI) getBalance(address string){
	bc := GetBlockchain()
	defer bc.db.Close()

	utxos := bc.FindUTXO(address)
	balance := 0
	for _, vout := range utxos{
		balance += vout.Value
	}

	fmt.Println("Balance of '%s' is %d", address, balance)
}

func (cli *CLI) send(from, to string, amount int){
	bc := GetBlockchain()
	defer bc.db.Close()

	utxo := NewUTXOTransaction(from, to, amount, bc)
	bc.MineBlock([]*Transaction{utxo})
	fmt.Println("success")
}

func (cli *CLI) createBlockchain(address string){
	bc := CreateBlockchain(address)
	bc.db.Close()
	fmt.Println("Done!\n")
}

func (cli *CLI) Run(){
	// cli.validateArgs()

	printChainCmd := flag.NewFlagSet("printchain", flag.ExitOnError)
	createBlockchainCmd := flag.NewFlagSet("createblockchain", flag.ExitOnError)
	getBalanceCmd := flag.NewFlagSet("getBalance", flag.ExitOnError)
	sendCmd := flag.NewFlagSet("send", flag.ExitOnError)
	
	createBlockchainAddress := createBlockchainCmd.String("address", "", "The address to send genesis block reward to")
	getBalanceAddress := getBalanceCmd.String("address", "", "The address to get balance for")
	sendFrom := sendCmd.String("from", "", "Source wallet address")
	sendTo := sendCmd.String("to", "", "Destination wallet address")
	sendAmount := sendCmd.Int("amount", 0, "Amount to send")

	switch os.Args[1] {
	case "printchain":
		err := printChainCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	case "createblockchain":
		err := createBlockchainCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	case "getbalance":
		err := getBalanceCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	case "send":
		err := sendCmd.Parse(os.Args[2:])
		if err != nil{
			log.Panic(err)
		}
	default :
		cli.printUsage()
		os.Exit(1)
	}

	if printChainCmd.Parsed(){
		cli.printChain()
	}
	if createBlockchainCmd.Parsed(){
		if *createBlockchainAddress == ""{
			createBlockchainCmd.Usage()
			os.Exit(1)
		}
		cli.createBlockchain(*createBlockchainAddress)
	}
	if getBalanceCmd.Parsed(){
		if *getBalanceAddress == ""{
			getBalanceCmd.Usage()
			os.Exit(1)
		}
		cli.getBalance(*getBalanceAddress)
	}
	if sendCmd.Parsed(){
		if *sendFrom == "" || *sendTo == "" || *sendAmount <= 0 {
			sendCmd.Usage()
			os.Exit(1)
		}

		cli.send(*sendFrom, *sendTo, *sendAmount)
	}
}
~~~~~

账户余额就是由账户地址锁定的所有未花费交易输出的总和。
在挖出创世块以后，来检查一下我们的余额：

![smiley](\assets\images\usedInBlogs\blockChainGo\4\4.png)

# 发送币

现在，我们想要给其他人发送一些币。为此，我们需要创建一笔新的交易，将它放到一个块里，然后挖出这个块。之前我们只实现了 coinbase 交易（这是一种特殊的交易），现在我们需要一种通用的普通交易：


~~~go
func NewUTXOTransaction(from, to string, amount int, bc *Blockchain) *Transaction {
	var inputs []TXInput
	var outputs []TXOutput
	var acc = 0

	txs := bc.FindUnspentTransactions(from)
	for _, tx := range txs {
		for txid, out := range tx.Vout {
			if out.CanBeUnlockedWith(from) && acc < amount{
				acc += out.Value
				input := TXInput{tx.ID, txid, from}
				inputs = append(inputs, input)
			}
		}
	}

	if acc < amount{
		log.Panic("ERROR: Not enough funds")
	}

	output := TXOutput{amount, to}
	outputs = append(outputs, output)

	if acc > amount{
		output.Value = acc - amount
		output.ScriptPubKey = from
		outputs = append(outputs, output)
	}

	tx := Transaction{nil, inputs, outputs}
	tx.SetID()
	return &tx
}
~~~~

在创建新的输出前，我们首先必须找到所有的未花费输出，并且确保它们有足够的价值（value）。随后，对于每个找到的输出，会创建一个引用该输出的输入。接下来，我们创建两个输出：

* 一个由接收者地址锁定。这是给其他地址实际转移的币。
* 一个由发送者地址锁定。这是一个找零。只有当未花费输出超过新交易所需时产生。记住：输出是不可再分的。

如果引用的输入大于支出，那么需要多加一笔输出到转出账户，这一步可以视为找零。

现在，我们可以创建 Blockchain.MineBlock 方法：

~~~go

func (bc *Blockchain) MineBlock(txs []*Transaction) {
	var prvHash []byte

	err := bc.db.View(func(tx *bolt.Tx) error{
		b := tx.Bucket([]byte(blocksBucket))
		prvHash = b.Get([]byte("1"))

		return nil
		})
	if err != nil{
		log.Panic(err)
	}
	
	block := NewBlock(txs, prvHash)

	err = bc.db.Update(func(tx *bolt.Tx) error{
		b := tx.Bucket([]byte(blocksBucket))
		err := b.Put(block.Hash, block.Serialize())
		if err != nil{
			log.Panic(err)
		}

		err = b.Put([]byte("1"), block.Hash)
		if err != nil{
			log.Panic(err)
		}

		bc.tip = block.Hash

		return nil
		})
	if err != nil{
		log.Panic(err)
	}
}
~~~~

最后，让我们来实现 send 方法：

~~~go
func (cli *CLI) send(from, to string, amount int){
	bc := GetBlockchain()
	defer bc.db.Close()

	utxo := NewUTXOTransaction(from, to, amount, bc)
	bc.MineBlock([]*Transaction{utxo})
	fmt.Println("success")
}
~~~~~


发送币意味着创建新的交易，并通过挖出新块的方式将交易打包到区块链中。不过，比特币并不是一连串立刻完成这些事情（虽然我们目前的实现是这么做的）。相反，它会将所有新的交易放到一个内存池中（mempool），然后当矿工准备挖出一个新块时，它就从内存池中取出所有交易，创建一个候选块。只有当包含这些交易的块被挖出来，并添加到区块链以后，里面的交易才开始确认。


让我们来检查一下发送币是否能工作：

![smiley](\assets\images\usedInBlogs\blockChainGo\4\5.png)

# 参考

本章节代码参考：https://github.com/Jeiwan/blockchain_go/tree/part_4

调试命令参考：

> blockChain  createblockchain -address Ivan
> blockChain getbalance -address Ivan
> blockChain send -from Ivan -to Pedro -amount 6
> blockChain  getbalance -address Ivan
> blockChain getbalance -address Pedro
