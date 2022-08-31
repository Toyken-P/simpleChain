# 代码注解

## 区块(block)

### 区块结构

实际是区块头内容

```go
type Block struct {
	Height        int64
	Timestamp     int64
	Transactions  []*Transaction
	PrevBlockHash []byte
	Hash          []byte
	Nonce         int
}
```

### 区块生成

```go
// 区块生成，构造 NewProofOfWork 并调用其方法挖掘区块
func NewBlock(transactions []*Transaction, prevBlockHash []byte, height int64) *Block
// 创世区块生成，调用 NewBlock 函数
func NewGenesisBlock(coinbase *Transaction) *Block
```

### 区块编码

```go
// 使用 gob 对区块编码和解码
func (b *Block) Serialize() []byte
func DeserializeBlock(d []byte) *Block
```



## 交易(Transaction)

### 交易结构

```go
type Transaction struct {
	ID   []byte
	Vin  []TXInput
	Vout []TXOutput
}
```

### 交易生成

```go
// 挖矿奖励交易(CoinbaseTX)，input 为空，调用 NewTXOutput 生成 output
func NewCoinbaseTX(to, data string) *Transaction
// 交易生成
// 1.根据 address 找到钱包对应公钥 PublicKey
// 2.调用 FindSpendableOutputs 找到满足交易额的未花费 Outputs
// 3.构造 inputs 和 outputs
// 4.调用 SignTransaction 对交易签名
func NewUTXOTransaction(from, to string, amount int, bc *Blockchain) *Transaction
```

### 交易处理

```go
// 对 Transaction 中的每个TXI进行单独地签名
func (tx *Transaction) Sign(privKey ecdsa.PrivateKey, prevTXs map[string]Transaction)
```

### 交易查找

```go
// 遍历所有 block，返回包含 UTXO 的 transaction
func (bc *Blockchain) FindUnspentTransactions(pubKeyHash []byte) []Transaction
// 新交易创建时，调用 FindUnspentTransactions 找到满足要求的可以供消费的交易
func (bc *Blockchain) FindSpendableOutputs(pubKeyHash []byte, amount int) (int, map[string][]int)
// 调用 FindUnspentTransactions 返回指定公钥hash值的所有UTXO，用于计算余额
func (bc *Blockchain) FindUTXO() map[string]TXOutputs
// 根据交易ID遍历所有 block 查找交易
func (bc *Blockchain) FindTransaction(ID []byte) (Transaction, error)
```



## 输入(TXInput) 和输出(TXOutput)

### 构造

```go
type TXInput struct {
	Txid      []byte
	Vout      int
	Signature []byte
	PubKey    []byte
}

type TXOutput struct {
	Value      int
	PubKeyHash []byte
}
```



### TXInput方法

```go
func (in *TXInput) UsesKey(pubKeyHash []byte) bool
```



### TXOutput方法

```go
// 为output进行签名，设置为Public Key Hash
func (out *TXOutput) Lock(address []byte)
// 检查output的PubKeyHash是否和用户的pubKeyHash一致
func (out *TXOutput) IsLockedWithKey(pubKeyHash []byte) bool
// 创建 TXOutput,调用 Lock 方法签名
func NewTXOutput(value int, address string) *TXOutput
```



## UTXO方法

```go
// 重构 UTXO set
// 若UTXO集合存在，将其删除；然后获取所有UTXO；最后将其保存到bucket中
func (u UTXOSet) Reindex()
// 每挖到区块后即更新UTX删除已消费的TXO，同时将新生成的交易中的UTXO添加进来
// 若某个交易的UTXO全部被删除，该交易也会随之被移除
func (u UTXOSet) Update(block *Block)
```





## Merkle Tree

### Merkle Tree结构

```go
type MerkleTree struct {
   RootNode *MerkleNode
}

type MerkleNode struct {
   Left  *MerkleNode
   Right *MerkleNode
   Data  []byte
}
```



### Merkle Node 生成

```GO
// 生成 Merkle Tree 节点
// 若 left，right 均为空，则创建叶子节点，data 保存交易 hash
// 否则将 left，right 节点数据相加后保存对应 hash
func NewMerkleNode(left, right *MerkleNode, data []byte) *MerkleNode
```



### Merkle Tree 生成

```go
// 创建 MerkleTree
func NewMerkleTree(data [][]byte) *MerkleTree
```





## 网络

### version

version 用于发现一个更长的 blockchain。当一个节点收到 version 消息后，将会检查发送节点的version.BestHeight 值是否更大（即 blockchain 更长），如果发送节点的 blockchain 更长的话，接收节点将会发送请求获取缺失的 block

```go
type version struct {
    Version    int	// 版本
    BestHeight int	// 本节点区块链长度
    AddrFrom   string	// 消息发送者地址
}
```



```go
var nodeAddress string
var knownNodes = []string{"localhost:3000"}

func StartServer(nodeID, minerAddress string) {
    nodeAddress = fmt.Sprintf("localhost:%s", nodeID)
    miningAddress = minerAddress
    ln, err := net.Listen(protocol, nodeAddress)
    defer ln.Close()

    bc := NewBlockchain(nodeID)

    if nodeAddress != knownNodes[0] {
        sendVersion(knownNodes[0], bc)
    }

    for {
        conn, err := ln.Accept()
        go handleConnection(conn, bc)
    }
}
```



### getblocks

返回当前节点所拥有的 block 的 hash 值列表，而不是所有 block 的详细信息。出于降低网络负荷的目的，若需要下载 block，可以从多个节点同时下载，没必要从单个节点下载

```go
type getblocks struct {
   AddrFrom string
}
```

调用 bc.GetBlockHashes() 获取区块 hash 列表，调用 sendInv 发送消息

```go
func handleGetBlocks(request []byte, bc *Blockchain)
```

### Inv

inv 包含发送节点所拥有的 Block 或交易的 hash 值列表。Type 用于表示是 Block 还是交易

```go
type inv struct {
	AddrFrom string
	Type     string
	Items    [][]byte
}
```



根据消息中的数据类型，返回 block 或者交易

```go
func handleGetData(request []byte, bc *Blockchain)
```



### getdata

```go
type getdata struct {
    AddrFrom string
    Type     string
    ID       []byte
}
```



根据消息中的数据类型，返回block或者交易

```go
func handleGetData(request []byte, bc *Blockchain)
```



### block和tx

```go
type block struct {
    AddrFrom string
    Block    []byte
}

type tx struct {
    AddFrom     string
    Transaction []byte
}
```

当收到一个新 block 时，将该 block 加入 blockchain 中。如果还有其他待下载的 block，将向相同的节点发送消息来获取 block 信息。当所有 block 均已下载，UTXO 将被重建。

```go
func handleBlock(request []byte, bc *Blockchain)
```



```go
func handleTx(request []byte, bc *Blockchain)
```
