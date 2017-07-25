---
title: Dynamo - A flawed architecture
date: 2017-07-18 08:35:12
tags: architecture
---

## Author

Joydeep Sen Sarma，印度人，07-11在facebook担任Data Infrastructure Lead，目前在一个创业公司任CTO
在facebook期间，从0搭建了数千节点的hadoop集群，也是Cassandra的core committer，参与开发Hive
https://www.linkedin.com/in/joydeeps/

编写在2009年，Dynamo paper发布于2007

## 'Flaws'

### R+W>N

作者主要以Cassandra的实现来评论Dynamoc论文，从而忽略了vector clock逻辑时钟来跟踪数据版本号来检测数据冲突
因此，遭到很多围观群众的声讨

作者的主要思想是：由于hinted handoff，缺少central commit log，缺少resync/rejoin barrier，transient failure会导致stale read

Cassandra解决办法，是尝试通过central commit log，那就变成了Raft了
https://issues.apache.org/jira/browse/CASSANDRA-225

作者如果提到下面的场景，可能更有说服力
```
// W=3 R=3 N=5(1,2,3,4,5)

Op            W           R
====        =====       =====
a=5         1,2,3   
                        3,4,5 // 由于有节点overlap，写入的数据可以读出来
2,3crash    
del(a)      1,4,5
2,3recover
                        1,2,3 // 2,3读出a=5，而1没有key(a)，如何处理冲突?
```

此外，由于sloppy quorum，下面的场景R+W>N也一样无法保证一致性
```
// W=3 R=3 N=5(1,2,3,4,5)
ring(A, B, C, D, E, F, G, A)

client set(k1)=3 // md5(k1)，发现它的coordinator是A，client send req to A
A会先写本地磁盘，然后并发向B,C复制，成功后，return to client ok
现在A, B, C全部crash
set(k1)=8 // 由于hinted handoff，它的coordinator变成D
D写盘、复制到E,F，ok。// 当然D,E,F里的数据有hint metadata，表示他们只是代替A,B,C
                   // 等A,B,C活，再transfer to them and cleanup local store
然后A, B, C全部recover
get(k1)，读到是3，而不是8
// 因为D,E,F的检测A/B/C活以及transfer hinted data是异步的
```

### WAN

如果一个node crash，'hinted handoff'会把数据写入一致性哈希环上的下一个node：可能是另外一个DC node
等crashed node恢复，如果网络partitioned，这个node会很久无法赶上数据，直到partition解除

但preference list里，已经考虑到了，每个key都会复制到不同的机房

### 论文里的"facts"

- 论文提到，商业系统通常通过同步复制保证一致性
  大部分数据库的复制都是异步的
- 论文提到，中心化架构降低availability
  不对，中心化会造成扩展瓶颈，并不会降低可用性，SPOF有非常多的方法解决

## Werner Vogels的回复

Dynamo SOSP论文有2个目的
- 展示如何利用多种技术创建一个生产系统
- 对学术界的一个反馈，学术到生产环境遇到的困难和解决办法

本论文不是一个blueprint，拿它就可以做出一个Dynamo clone的

我认为我的论文真正的贡献，是让人设计系统时的权衡trade-off

## Dynamo回顾

Java实现的，通过md5(key)进行partition，由coordinator node复制read/write/replicate到一致性哈希虚拟节点环上
gossip进行membership管理和健康检查，提出了R/W/N的quorum模式

- no updates are rejected due to failures or concurrent writes
- zero-hop DHT
  为了latency，每个node有全局的路由信息，不产生hop
- resolve update conflicts，在read时由应用做，write时不检测
  - 这与大部分系统相反，原因是为了always writeable
  - read时，让应用处理冲突，而不是Dynamo store做
    store做，由于抽象层，只能简单的类似last win的策略
    应用更懂，可以做类似union等高级处理策略
- md5(key) => target nodes
  hash conflict? 可以不用考虑，无非就是让一台机器多处理了一个key而已
- vector clock
  [(node, counter), (node, counter), ...]
  get(key)返回的ctx，在put(key, ctx, val)时会带过去：ctx里包含该key的vector clock信息
- R W N
  get/put latency都取决于R/W里最慢节点的latency
- get(key)
  如果没有冲突，返回一个值；发现冲突，返回list
  返回的每个value都有对应的一个context(vector clock version)
- replication
  - write coordinator
    put(key, ctx, val)根据md5(key)计算出coordinator node，它负责写入本地磁盘，同时向顺时针后面的N-1个alive节点进行复制
    由于后面N-1节点可能come and go，它是动态的，coordinator盲目地找出后面活着的：这样才能有高可用 sloppy quorum
- coordinator and replication
  负责生成新的vector clock，并把(key, val, vc)写入本地
  同时向顺时针后面的N-1个healthy节点进行复制(dead nodes跳过)
  但由于虚拟节点的存在，需要保证复制到的节点不在一台机器上
  preference list是一个key对应的storage node，在ring上，如果顺时针发现有机器重叠就忽略并继续顺时针找
- 节点的加入和退出
  手动显示
- local persistence engine
  插件架构，amazon主要使用的是BerkelayDB，但也可以使用mysql/BDB Java等
- SLA
  99.9% 300ms

### Replication

每个key都被复制在N台机器(其中一台是coordinator)，coordinator向顺时针方向的N-1个机器复制
preference list is list of nodes that is responsible for storing a particular key，通常>N，它是通过gossip在每个节点上最终一致
```
okN = 0
for node = range preferenceList {
    // 论文中没有提到复制是顺序还是并发
    // 如果顺序，latency是个问题
    // 如果并发，为了HA和latency，>N个节点会进行复制，造成空间浪费
    if replicateTo(node) == ok {
        okN++
    }

    // all read/write are performed on the first N healthy nodes from pref list
    if okN == N {
        break
    }
}
```

Merkle树
```
node ring(A, B, C, D, E, A), W=3
A crash
那么本来应该复制到A的数据，会复制到D
由于membership change是显示的，D知道这个数据是它临时替A代管的，它会写到临时存储，并携带hint(A) meta info，并定期检查A是否recover
如果A ok，D会把本来该写到A的数据transfer给A，成功后，本地删除

但，如果D永久crash，而A recover，那么这些hinted data，A就无从transfer了
此时，通过Merkle进行检查、增量同步
但它的问题是当有node进入、离开时，这个树要重新创建，在key多的时候非常费时，因此，它只能异步后台执行
```

### IDCs

只是在preference list上配一下，架构上并没有过多考虑
仍然是coordinator负责复制，首先本地机房，然后远程机房

## References

http://jsensarma.com/blog/?p=55
http://jsensarma.com/blog/?p=64
https://timyang.net/data/dynamo-flawed-architecture-chinese/
https://news.ycombinator.com/item?id=915212
http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf
