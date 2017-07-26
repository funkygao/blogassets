---
title: GFS & Colossus
date: 2017-07-25 14:39:34
tags: storage
---

## GFS

GFS，2001年开发出来，3个人，在部署前用了1年，论文发表于2003
BigTable，2003年开发出来

### master

- replicated operation(redo) log with checkpoint to shadow masters
  - 当时没有自动failover，全是人工操作DNS alias
  - read请求可以使用shadow masters
  - 生成checkpoint时，master切换到新的redo log file，创建新线程生成checkpoint
    在此过程中发生的变化，记录到新redo log file
- bottleneck?
  - 过高的OPS
    由于redo log需要复制，master的tps也就在1万左右
    - client cache
      何时invalidate and recall master?
      - primary unreachable
      - primary reponse: no lease
    - client batch request
  - 内存容量
    - prefix compression to reduce memory footprint
    - 每个chunk(64MB)，metadata占内存64B
      如果保存1百万个文件(64TB)，需要64MB内存
      如果保存10亿个文件(64PB)，需要64GB内存
      实际上是到了5千万个文件，10PB的时候，master已经到达瓶颈了

### Replication

- operation log between master and shadows
- client write data flow to pipelined chain chunkservers
- primary write control flow to secondary chunkservers

### 磁盘错误的解决

file -> chunk -> block

在每个chunkserver上执行，通过checksum检测

每个chunk由多个64KB的block组成，每个block有一个32位的checksum

read repair，如果chunkserver发现了一个非法block，会返回client err，同时向master汇报
- client会从其他replica read
- master会从其他有效的replica把这整个chunk clone到另外一个chunkserver，然后告诉有问题的chunkserver删除那个chunk

### Questions

#### chunk为什么64MB那么大?

- 减少master内存的占用
- 减少client与master的交互
  同一个chunk的R/W，client只需要与master交互一次
- 可以很容易看到机器里哪些机器磁盘快满了，而做迁移
- 可以减少带宽的hotspot
  如果没有chunk，那么1TB的文件就只能从一个replica读
  有了chunk，可以从多个replica读
- sharding the load
- 加快recovery时间
  每个chunk，可以很快地被clone到一台新机器
- 如果一个file只有1MB，那么实际存储空间是1MB，而不是64MB
- 可以独立replicate
  - 文件的一部分损坏，可以迅速从好的replica恢复
  - 支持更细粒度的placement
- 支持超大文件

#### Data flow为什么pipelined chain，而不并发?

为了避免产生网络瓶颈，同时为了更充分利用high latency links
通过ip地址，感知chunkserver直接的距离

Total Latency = (B/T) + (R*L)

```
Phase1: data flow  client -> chained chunkservers
client由近及远地chain把数据写入相应chunkserver的LRU buffer
这个顺序跟primary在哪无关

Phase2: control flow  client -> primary -> secondary chunkservers
确定mutation order
```

## 2009 GFS回顾

### GFS的问题

- 扩展性：在50M文件，10PB的时候出现瓶颈
- 有需要不那么大文件的应用
- 有对于latency敏感的应用

## Colossus

Colossus is specifically designed for BigTable, 没有GFS那么通用
In other words, it was built specifically for use with the new Caffeine search-indexing system, and though it may be used in some form with other Google services, it is not the sort of thing that is designed to span the entire Google infrastructure.

- automatically sharded metadata layer
- EC
- client driven replication
- metadata space has enabled availability analysis


## References

http://queue.acm.org/detail.cfm?id=1594206
http://google-file-system.wikispaces.asu.edu/
