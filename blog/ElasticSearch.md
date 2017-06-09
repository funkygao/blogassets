---
title: ElasticSearch
date: 2017-06-05 13:35:53
tags: storage
---

默认1s会把MemoryBuffer写到new segment，但fsync
![write](https://github.com/funkygao/blogassets/blob/master/img/eswrite1.jpg?raw=true)
![read](https://github.com/funkygao/blogassets/blob/master/img/esread1.jpg?raw=true)

shard = MurMurHash3(document_id) % (num_of_primary_shards)

index segements merge

Node.start()

replication PUSH model

Node1节点用数据的_id计算出数据应该存储在shard0上，通过cluster state信息发现shard0的主分片在Node3节点上，Node1转发请求数据给Node3,Node3完成数据的索引
Node3并行转发数据给分配有shard0的副本分片Node1和Node2上。当收到任一节点汇报副本分片数据写入成功以后，Node3即返回给初始的接受节点Node1，宣布数据写入成功。Node1成功返回给客户端。

consistency:默认主分片在尝试写入时需要规定数量(quorum)或过半的分片(可以使主分片或复制分片)写入成功，就返回给客户端。consistency允许为one(只有一个主分片，与上面的replication等同)
