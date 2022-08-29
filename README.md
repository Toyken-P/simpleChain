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
// 根据 data 序列创建 MerkleTree
func NewMerkleTree(data [][]byte) *MerkleTree
```
