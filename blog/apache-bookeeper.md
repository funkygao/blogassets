---
title: apache bookeeper
date: 2017-05-23 17:04:32
tags: PubSub
---

## Features

- 没有topic/partition概念，一个stream是由多个ledger组成的，每个ledger是有边界的
  createLedger(int ensSize, int writeQuorumSize, int ackQuorumSize)
- ledger只有int id，没有名字
- 每个entry(log)是有unique int64 id的
- striped write: 交错存储
- 各个存储节点bookie之间没有明确的主从关系
- shared WAL
- single writer
- Quorum投票复制，通过存储在zk里的LastAddConfirmedId确保read consistency
- bookie does not communicate with other bookies，由client进行并发broadcast/quorum

## VS Kafka

![vs kafka](https://github.com/funkygao/blogassets/blob/master/img/kfk_bk.jpg?raw=true)

createLedger时，客户端决定如何placement(org.apache.bookkeeper.client.EnsemblePlacementPolicy)，然后存放在zookeeper
例如，5个bookie，createLedger(3, 3, 2)

## IO Model

![io](https://github.com/funkygao/blogassets/blob/master/img/bk_io.jpg?raw=true)

- 读写分离
- Disk 1: Journal(WAL) Device
  - {timestamp}.txn
- Disk 2: Ledger Device
  - 数据存放在多个ledger目录
  - LastLogMark表示index+data在此之前的都已经持久化到了Ledger Device，之前的WAL可以删除
  - 异步写
  - 而且是顺序写
    - 所有的active ledger共用一个entry logger
    - 读的时候利用ledger index/cache
- [Disk 3]: Index Device
  - 默认Disk2和Disk3是在一起的
- 在写入Memtable后，就可以向client ack了

### IO被分成4种类型，分别优化

- sync sequential write: shared WAL
- async random write: group commit from Memtable
- tail read: from Memtable
- random read: from (index + os pagecache)

## References

https://github.com/ivankelly/bookkeeper-tutorial
https://github.com/twitter/DistributedLog
