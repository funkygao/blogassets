---
title: blockchain
date: 2017-09-11 08:25:49
tags: algorithm
---

## Overview

目前公开的bitcoin只能支持每秒7笔的吞吐量，一般对于大额交易来说，安全的交易确认时间为1小时(6个block生成时间)

中本聪设计比特币时，为区块设置了1MB的容量限制，使每一个区块只能容纳4096个交易；同时，工作量证明机制使得确认交易并将交易记录到区块链中需要约10分钟

bitcoin是一种通货紧缩货币，到2140年到达最大值：2100万个比特币

blockchain的所谓去中心化，实际上是建立一个少数服从多数的分布式权威网络，只要这个网络上节点足够多，就是可以信赖的
以前，用户trust银行；现在，用户trust majority nodes in blockchain network

Bitcoin accounts: address
public key: 1MKe24pNsLmFYk9mJd1dXHkKj9h5YhoEey
private key: 5KkKR3VAjjPbHPzi3pWEHVQWrVa3C4fwD4PjR9wWgSV2D3kdmeM

Currently, the top 10 mining pools consistently create about 90% of the blocks, and China-based pools create more than 60% of the blocks. 
所以，虽然bitcoin设计上是去中心的，但目前的现状是实际上控制在少数人手里

![miners](https://bitsonblocks.files.wordpress.com/2015/09/miner_dist_2.jpg?w=594&h=476&crop=1)

### bitcoin

- Blocks 
  481,823
- Total BTC   
  16.523M
- Difficulty 
  923233068449
- Network total 
  7,983,858 trillion hash per second
- Blocks/hour   
  7.25

## Key Design

BlockChain是以牺牲效率和性能为代价但换来了可靠的安全的最终一致性的分布式数据库
- 与数据库不同，它不保存状态，只保存commit log，而通过commit log来计算出状态
  bitcoin的blockchain里并不保存balance，只有[]transaction(inputs, outputs, sender, receiver)
- no range query
- blockchain可以想象成只有1个table的replicated database
- each transaction must commit before next transaction can enter proposal phase
  - throughput bottleneck

关键技术
- P2P
  节点发现和通信
- hash, 非对称加密
  sender: sign(privatekey_of_sender, publickey_of_receiver)
- merkle tree
- wallet app

### Block

![blockchain](https://bitsonblocks.files.wordpress.com/2015/09/bitcoin_blockchain_infographic1.jpg)

#### Merkle Tree

每一个block有一个merkle树，叶子节点是transaction，每个中间节点有2个指针(2叉树)
如果修改某一个transaction叶子，就会传递到tree root

merkle tree，目前还没有在bitcoin中使用，但它的作用是：
确认一个交易是否被confirm，只要下载block header+merkle tree，而不用下载整个block

### Proof of Work

sha256('pay me, joe, 25 bitcoins. 000001') => 4699f0e443b73ddd6fdc14d4662395361fa21f54e647c64e643a49c54fef511c
sha256('pay me, joe, 25 bitcoins. 000013') => 0ac09823cf2309c63d425c21b1c3d83f6dbc4b8acfb80b37b2db3d544192b7e9

猜到第13个nonce，就得到1个0开头的hash
bitcoin目前的难度是需要14个0开头，实际上每增加1个0，就把原来的计算量提高了16倍

bitcoin sha256(input + nonce)，其中input包括
- computer code
- receiving address(public key)
- transactions
- timestamp
- prev hash

### Sybil Attack

女巫攻击，来源是70年代的一部叫做《Sybil》的美国系列片。片中的女主角人格混乱，扮演着16个角色。

### Transaction

```
1. client send payment to replica nodes, proposing new log entries
2. proposal会在replica nodes间进行广播(unconfirmed transaction)，进行leader election: proposal phase
   transaction临时保存在node(miner)的内存
3. 通过PoW，leader胜出，本地append到blockchain，并广播到其他replicas
   miner之间是竞争关系，都在正确尽快solve the puzzle to be leader
   miner会把unconfirmed transactions进行合并(batch)，生成到一个block(一个block最多4096个transactions)
4. 所有replicas append to local blockchain，transaction confirmed

block chain实际上实现了global transaction time order
```

### Block Acceptance

收到miner发来的新block后，node会very block, verify each transaction, verify not spent
都成功，就加入local blockchain

如果local blockchain没有与main blockchain同步，那么verify block会失败，它会通过P2P请求同步，把unsynced blocks追上：

```
node1: block 1, 2, 3, 4
main blockchain: 1, 2, 3, 4, 5, 6, 7
此时node2成功创建了block8，并广播给node1
node1发现block 5-7还没有同步过来，先进行同步，然后再把block8 append to local blockchain
```

### Confirm

确认数 = 该交易所在的block在blockchain中后面的block数量 + 1
如果确认数 >= 6，那么就可以认为无法逆转了

### Consensus

通过PoW，每次生成block时进行一次leader选举，由leader生产new block，猜数成功的node自认为成为了leader，然后通过P2P广播(gossip)
由于猜数比较困难，多节点同时成为leader并且在接收到其他leader广播前猜成功的可能性非常小，但还是存在可能性，即多主，这就需要解决冲突

miner一旦recv a new block，就意识到在这个block的race上自己输了，它会立即把一些pending transactions和这个收到的block作为prev hash来构建下一个block

### Conflict Resolve

采用longest chain rule，发现冲突时，block chain fork branch，在发现longest chain时，把short chain删除：但可能会造成confirmed transaction lost

![conflict](https://github.com/funkygao/blogassets/blob/master/img/longrule1.png?raw=true)
![resolve](https://github.com/funkygao/blogassets/blob/master/img/longrule2.png?raw=true)

### Double Spent

![double-spent](https://github.com/funkygao/blogassets/blob/master/img/double_spend1.jpg?raw=true)

### Public keys as identities

Alice pays Bob，Alice会把这个交易通过private key进行签名，广播给miners
- Bob不需要参与
- 交易信息里只包含Alice和Bob的public keys(address)
- You can create a new identity at any time by generating a new key pair
  with no central authority or registry

### 验证身份verify

![signature](https://github.com/funkygao/blogassets/blob/master/img/digital_signature.png?raw=true)

比特币的所有权是通过数字密钥、比特币地址、数字签名来确立的
密钥实现了去中心化的信任，所有权认证

创建交易时，payer利用自己的private key给transaction签名，同时把自己的public key存放在消息里：(payer_public_key, signature)
miners通过这个信息，就可以verify这个transaction确实是payer发出的
同时，transaction里也包含了payee的public key信息，只有payee利用他的private key才能解开

key pair，非对称加密的特性：
- 用私钥加密的内容，只能用公钥解密
  如果能通过(payer_public_key, signature)能解密，就证明了payer的身份
- 用公钥加密的内容，只能用私钥解密
  只有payee才能用他的私钥解密交易
  每个交易必须有效的签名才能被存入ledger
  当payee花销这笔钱时，他必须给这个交易签名，由于这笔钱已经在ledger里记录了payee的公钥，只有payee才能签名
- 公钥用于收钱，私钥用于花钱时生成数字签名
- 通过私钥能计算出公钥，但反过来不行
  只要私钥不丢，公钥丢了也没关系

如果我知道Alice的public key(X)，然后创建一笔交易：X支付给me 1BTC，系统是怎么知道有诈的?
首先我不知道X对应的私钥，只能拿我的私钥对交易加签名，miner通过X和signature就能验证：invalid signature

### Upgrade

https://github.com/bitcoin/bips

BIP(bitcoin improvement proposal)在github上发布，miners自愿地下载、启动，但不是强制的

## Q & A

### 为什么blocks通过hash进行chain，而不是通过普通的编号？

因为hash具有不可篡改性，hash本身包含了内容的指纹

### 同时有很多client进行交易，full node一直在瞎忙活？

T1, T2, Tn被分发到n个full node，那么每个node都开始猜数，其中一个node(N1)猜中，开始广播，此时N2~Nn还在为自己的Tn进行猜数，发现new block，就停下手中的活，重新生成新的block，并重新猜数
如果N1在广播时，消息还没有传到Nx，而此时Nx的猜数工作是不会停的；如果此时Nx也猜数成功，那么在它还没有收到N1广播前，就会广播自己的new block，此时fork出现
对于某一个full node，它可能会并发接收多个交易请求，它会进行串行化

### 如果识别一个account

通过wallet程序，每次交易都可以生成新的public/private key pair，由wallet管理并保存。如果wallet的数据丢了，那么bitcoin就无法证明是你的了

### 如何动态维护PoW的难度？

每个节点计算最近2016个solution，因为频率应该是10分钟一个，因此2016个，应该是2016/(6*24)=14天
而每个block里都有timestamp
```
elapsed = timestamp(block(-2016)) - timestamp(block(-1)) 
// 例如，假如elapsed = 7 day，那么说明难度系统太低了
// 即，移动平均数
difficulty = difficulty * 2
```

### 如何保证bitcoin address唯一？

Bitcoin address是由34个alphanumerical组成的字符串，但排除了O/T/l/0，它的空间是58^34，即
904798310844700775313327215140493940623303545877497699631104
但里面有几位用于checksum，它的实际空间是2^160，即
1461501637330902918203684832716283019655932542976

### 一个恶意node可以做什么，不能做什么？

它能
- Refuse to relay valid transactions to other nodes
  但其他节点会replay
- Attempt to create blocks that include or exclude specific transactions of his choosing
- Attempt to create a ‘longer chain’ of blocks that make previously accepted blocks become ‘orphans’ and not part of the main chain

它不能
- Steal bitcoins from your account
- Make payments on your behalf or pretend to be you


### 为什么建议每次transaction都新生成key pair？

公钥虽然没有安全问题，但有隐私，毕竟它属于某一个人，如果我的交易都用一个key pair，那么最终这些交易可以发现是一个人的行为

### 如果写错了接受者payee公钥，这笔钱能回来吗？

首先，公钥(bitcoin address)是有校验位的，写错的时候，基本上就可以在提交时离线发现
如果恰巧校验一致，而地址是不存在的，那么交易会成功，但由于那个payee不存在，也就不存在对应的private key，也就无法spend it：黑洞，那笔钱永远消失了

如果本来我想给A钱，却输入时写成了B的address，那么：
Bitcoin transactions are not reversible. Sending to the wrong person cannot be undone.

### Block timestamp如果做假怎么办？

A timestamp is accepted as valid if it is greater than the median timestamp of previous 11 blocks, and less than the network-adjusted time + 2 hours. 
"Network-adjusted time" is the median of the timestamps returned by all nodes connected to you.

Whenever a node connects to another node, it gets a UTC timestamp from it, and stores its offset from node-local UTC. The network-adjusted time is then the node-local UTC plus the median offset from all connected nodes. Network time is never adjusted more than 70 minutes from local system time, however.

### 如果真有人控制了51%计算能力，会发生什么？

attacker只能把他刚花的钱payment取消掉，即double spent

### 据说bitcoin最大吞吐量是7 TPS，怎么计算来的？

每个block最大1MB，4096个transaction，10分钟产生一个block

4096/(10*60) = 6.83 = 7

### miner做恶的惩罚

miner竞争成功后，创建一个block，它会获得奖励；如果它发布一个非法的block，那么大部分miners会拒绝；并在下一个block时，该非法block被取消，同时它的奖励也被取消。除非，它能保证它一直是竞争成功

### seed nodes?

hard coded
https://github.com/bitcoin/bitcoin/blob/863e995b79ec388bf292d80f181912d01e20e2e5/src/net.cpp#L1198

```
unsigned int pnSeed[] =
{
    0x959bd347, 0xf8de42b2, 0x73bc0518, 0xea6edc50, 0x21b00a4d, 0xc725b43d, 0xd665464d, 0x1a2a770e,
    0x27c93946, 0x65b2fa46, 0xb80ae255, 0x66b3b446, 0xb1877a3e, 0x6ee89e3e, 0xc3175b40, 0x2a01a83c,
    0x95b1363a, 0xa079ad3d, 0xe6ca801f, 0x027f4f4a, 0x34f7f03a, 0xf790f04a, 0x16ca801f, 0x2f4d5e40,
    0x3a4d5e40, 0xc43a322e, 0xc8159753, 0x14d4724c, 0x7919a118, 0xe0bdb34e, 0x68a16b2e, 0xff64b44d,
    // 列出了500多个节点
}
```

## References

https://bitcoincharts.com/
https://blockchain.info/
https://en.bitcoin.it/wiki/Vocabulary
http://www.cs.rice.edu/Conferences/IPTPS02/101.pdf
https://www.cryptocompare.com/wallets/guides/how-to-create-a-bitcoin-address-from-a-public-key/
