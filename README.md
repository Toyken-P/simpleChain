# Blockchain in Go

## 代码注释
### block.go

### proofwork.go
```go
target.Lsh(target, uint(256-targetBits))  // 将target的值左移指定位
prepareData // 将数据装入区块准备计算hash
```

### blockchain.go
tip：最新区块的hash，用于指向数据库中最新区块的位置
```go
// 为避免运行过程中数据库被反复打开，因此在blockchain存储已打开的数据库链接
type Blockchain struct {
	blocks []*Block
	db     *bolt.DB
}
// bucket是与键值对的集合。
// 使用只读事务获取当前数据库中最新block的hash值
err := bc.db.View(func(tx *bolt.Tx) error {
b := tx.Bucket([]byte(blocksBucket))
lastHash = b.Get([]byte("l"))

// 将区块hash作为key，序列化后的内容作为value
err := b.Put(newBlock.Hash, newBlock.Serialize())
```

### blockchain_iterator.go
我们想按照block被加入的顺序来输出blockchain内容，但bucket中的key是按照字节序的顺序来存储的。
此外，我们不想将所有block全部加载到内存，因此将通过迭代器来逐个遍历block

一开始迭代器指向blockchain的tip，意味着将从顶到底、从新到旧的遍历blockchain。
选择tip就好比blockchain投票。Blockchain可能有多个分支，最长的那个分支被当做主分支。
获得tip后，就可以重建整个blockchain并且得到blockchain的长度。因此，某种程度上可以说tip是blockchain的标识。

### transaction.go
coinbase交易的TXI不会引用任何TXO，而会直接生成一个TXO奖励给矿工
```go
// ID 为transaction本身编码后的hash值
type Transaction struct {
ID   []byte
Vin  []TXInput
Vout []TXOutput
}
// TXO 收款方，存储货币信息（TXOutput的Value字段），同时通过一个puzzle进行锁定，puzzle存放在ScriptPubKey字段中。
// 一个TXO作为一个整体使用，是**不可分割**的
type TXOutput struct {
Value        int
ScriptPubKey string
}
// TXI 付款方，与之前的某个TXO相关联：Txid存储输出所属的交易的ID，Vout存储输出的序号（一个交易可以包括多个TXO）
type TXInput struct {
Txid      []byte
Vout      int
ScriptSig string
}

// coinbase仅有一个TXI，该TXI的Txid为空，Vout设置为-1，同时ScriptSig中存储的不是脚本，而仅仅是一个普通字符串。
func NewCoinbaseTX(to, data string) *Transaction{}

// 查找包含UTXO的交易,即未被任何TXI引用的TXO
func (bc *Blockchain) FindUnspentTransactions(address string) []Transaction{}
```

### wallet.go
根据公钥生成比特币地址
1.使用 RIPEMD160(SHA256(PubKey))对公钥进行两次哈希，生成pubKeyHash
2.追加版本信息到pubKeyHash之前，生成versionedPayload，此时versionedPayload=version+pubKeyHash
3.使用SHA256(SHA256(versionedPayload))进行两次哈希得到hash值，取该值的前n个字节生成checksum
4.将checksum追加到versionedPayload之后，生成编码前的地址，此时地址= version+pubKeyHash+checksum
5.使用Base58对version+pubKeyHash+checksum编码生成最终的地址。
```go

```



# Part 4 图像说明
## 区块数据结构
![image-20220817144655216](D:\code\simpleChain\README.assets\image-20220817144655216.png)

## 地址生成

![比特币地址生成](D:\code\simpleChain\README.assets\比特币地址生成.png)
