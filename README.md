# Blockchain in Go

A blockchain implementation in Go, as described in these articles:

1. [Basic Prototype](https://jeiwan.net/posts/building-blockchain-in-go-part-1/)
2. [Proof-of-Work](https://jeiwan.net/posts/building-blockchain-in-go-part-2/)
3. [Persistence and CLI](https://jeiwan.net/posts/building-blockchain-in-go-part-3/)
4. [Transactions 1](https://jeiwan.net/posts/building-blockchain-in-go-part-4/)
5. [Addresses](https://jeiwan.net/posts/building-blockchain-in-go-part-5/)
6. [Transactions 2](https://jeiwan.net/posts/building-blockchain-in-go-part-6/)
7. [Network](https://jeiwan.net/posts/building-blockchain-in-go-part-7/)

## 代码注释
### block.go
```go
strconv.FormatInt   // 将数值变量转换为指定进制后变换为字符串
bytes.Join  // 将传入的[][]byte中的各[]byte插入指定字符后拼接，返还为一个[]byte
```

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
实际上，选择tip就好比blockchain投票。Blockchain可能有多个分支，最长的那个分支被当做主分支。
获得tip后，就可以重建整个blockchain并且得到blockchain的长度。因此，某种程度上可以说tip是blockchain的标识。
```go

```