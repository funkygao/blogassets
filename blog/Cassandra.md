---
title: Cassandra vs ScyllaDB
date: 2017-05-31 08:16:23
tags: storage
---

## Cassandra

Cassandra 项目诞生于 Facebook，后来团队有人跳到 Amazon 做了另外一个 NoSQL 数据库 DynamoDB。

Dynamo论文发表于2007年，用于shopping cart
Cassandra在2008年被facebook开源，用于inbox search

### Features

- CQL(Cassandra Query Language)
- 1,000 node clusters
- multi-data center
- out-of-the-box replication
- ecosystem with Spark

### Internals

- LSM Tree
- Gossip P2P
- DHT
- consistency same as DynamoDB
  - ONE
  - QUORUM
  - ALL
  - read repair
- Thrift

## ScyllaDB

KVM核心人员用C++写的Cassandra(Java) clone，单机性能提高了10倍，主要原因是：
- DPDK, bypass kernel
- O_DIRECT IO, bypass pagecache, cache由scylla自己管理
  - pagecahce的格式必须是文件的格式(sstable)，而app level cache更有效，更terse
  - compaction的时候，pagecache讲是个累赘，它可能造成很多热点数据的淘汰
- 把一个node看做是多个cpu core组成的cluster, share nothing
- sharding at the cpu core instead of node
  更充分利用多核，减少contention，充分利用cpu cache, NUMA friendly
- 在需要core间交换数据时，使用explicit message passing
- avoid JVM GC

## References

https://db-engines.com/en/ranking
https://github.com/scylladb/scylla
http://www.scylladb.com/
http://www.seastar-project.org/
https://www.reddit.com/r/programming/comments/3lzz56/scylladb_cassandra_rewritten_in_c_claims_to_be_up/
https://news.ycombinator.com/item?id=10262719
