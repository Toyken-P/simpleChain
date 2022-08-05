# Blockchain in Go

A blockchain implementation in Go, as described in these articles:

1. [Basic Prototype](https://jeiwan.net/posts/building-blockchain-in-go-part-1/)
2. [Proof-of-Work](https://jeiwan.net/posts/building-blockchain-in-go-part-2/)
3. [Persistence and CLI](https://jeiwan.net/posts/building-blockchain-in-go-part-3/)
4. [Transactions 1](https://jeiwan.net/posts/building-blockchain-in-go-part-4/)
5. [Addresses](https://jeiwan.net/posts/building-blockchain-in-go-part-5/)
6. [Transactions 2](https://jeiwan.net/posts/building-blockchain-in-go-part-6/)
7. [Network](https://jeiwan.net/posts/building-blockchain-in-go-part-7/)

## 代码备注
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