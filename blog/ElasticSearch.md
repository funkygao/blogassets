---
title: ElasticSearch internals
date: 2017-06-05 13:35:53
tags: storage
---

## Basics

### Storage Model

一种特殊的LSM Tree，只是没有多level
```
translog   WAL
buffer     MemTable
segment    SSTable
merge      merge & compaction
```

### Defaults

```
node.master: true
node.data: true
index.number_of_shards: 5
index.number_of_replicas: 1
path.data: /path/to/data1,/path/to/data2
bootstrap.mlockall: true
http.max_content_length: 100mb
transport.tcp.port: 9300
```

## Indexing

![write](https://github.com/funkygao/blogassets/blob/master/img/eswrite1.jpg?raw=true)

- translog(WAL)
  - 每个shard一个translog，即每个index一个translog
  - 它提供实时的CRUD by docID，先translog然后再segments
  - 默认5s刷盘
  - translog存放在path.data，可以stripe，但存在与segments竞争资源问题
    ```
    $ls nodes/0/indices/twitter/1/
    drwxr-xr-x  3 funky  wheel   102B  6  8 17:23 _state
    drwxr-xr-x  5 funky  wheel   170B  6  8 17:23 index
    drwxr-xr-x  3 funky  wheel   102B  6  8 17:23 translog
    ```
- refresh
  - scheduled periodically(1s)，也可手工 /_refresh
  - 在此触发merge逻辑
  - refresh后，那么在此之前的所有变更就可以搜索了，in-memory buffer的数据是不能搜索的
  - ![refresh](https://github.com/funkygao/blogassets/blob/master/img/elarefresh.png?raw=true)
    - 产生新segment，但不fsync(close不会触发fsync)，然后open(the new segment)
    - 不保证durability，那是由flush保证的
    - in-memory buffer清除
- flush
  - 30s by default，会触发commit
  - Any docs in the in-memory buffer are written to a new segment
  - The buffer is cleared
  - A commit point is written to disk
  - The filesystem cache is flushed with an fsync
  - The old translog is deleted
- commit
  - 把还没有fsync的segments，一一fsync
  - ![commit](https://github.com/funkygao/blogassets/blob/master/img/elasticsearch-internals.png?raw=true)
- merge
  - ![merge](https://github.com/funkygao/blogassets/blob/master/img/elamerge.png?raw=true)
  - policy
    - tiered(default)
    - log_byte_size
    - log_doc
  - 可能把committed和uncommitted segments一起merge
  - 异步，不影响indexing/query
  - 合并后，fsync the merged big segment，之前的small segments被删除
  - ES内部有throttle机制控制merge进度，防止它占用过多资源: 20MB/s
- update
  delete, then insert

## Query

![read](https://github.com/funkygao/blogassets/blob/master/img/esread1.jpg?raw=true)

### local search

```
translog.search(q)
for s in range segments {
    s.search(q)
}
```

### deep pagination

search(from=50000, size=10)，那么每个shard会创建一个优先队列，队列大小=50010，从每个segment里取结果，直到填满
而coordinating node需要创建的优先队列 number_of_shards * 50010

## Cluster

### Intro

所有node彼此相连，n*(n-1)个连接
shard = MurMurHash3(document_id) % (num_of_primary_shards)

Node1节点用数据的_id计算出数据应该存储在shard0上，通过cluster state信息发现shard0的主分片在Node3节点上，Node1转发请求数据给Node3,Node3完成数据的索引
Node3并行转发数据给分配有shard0的副本分片Node1和Node2上。当收到任一节点汇报副本分片数据写入成功以后，Node3即返回给初始的接受节点Node1，宣布数据写入成功。Node1成功返回给客户端。

### Consensus

zen discovery(unicast/multicast)，存在脑裂问题，没有去解决，而是通过3台专用master机器，优化master逻辑，减少由于no reponse造成的partition可能性
```
node1, node2(master), node3
2-3不通，但1-2, 2-3都通，minimum_master_nodes=2，node3会重新选举自己成为master，而1同意了，2个master
```

默认ping_interval=1s ping_timeout=3s join_timeout=20*ping_interval

没有使用Paxos(zk)的原因:
>This, to me, is at the heart of our approach to resiliency. 
>Zookeeper, an excellent product, is a separate external system, with its own communication layer. 
>By having the discovery module in Elasticsearch using its own infrastructure, specifically the communication layer, it means that we can use the “liveness” of the cluster to assess its health. 
>Operations happen on the cluster all the time, reads, writes, cluster state updates.
>By tying our assessment of health to the same infrastructure, we can actually build a more reliable more responsive system. 
>We are not saying that we won’t have a formal Zookeeper integration for those who already use Zookeeper elsewhere. But this will be next to a hardened, built in, discovery module.

### Concurrency

每个document有version，乐观锁

### Consistency

- quorum(default)
- one
- all

### Replication

PUSH model

## References

https://www.elastic.co/blog/resiliency-elasticsearch
https://github.com/elastic/elasticsearch/issues/2488
http://blog.mikemccandless.com/2011/02/visualizing-lucenes-segment-merges.html
http://blog.trifork.com/2011/04/01/gimme-all-resources-you-have-i-can-use-them/
