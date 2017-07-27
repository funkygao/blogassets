---
title: replication models
date: 2017-07-19 19:12:26
tags: storage
---

## Models

- primary-replica(failover, replica readable)/multi-master/chain
- replica/ec
- push/pull
- sync/async

```
Dynamo      PUSH coordinator -> preference list next nodes in hash ring
Raft/ZK     PUSH master commit log -> majority ack
GFS/Hadoop  chain replication
ES          PUSH primary-replica model, master concurrently dispatch index req to all replicas
MySQL       PUSH master async PUSH to slaves binlog
Kafka       PULL(Fetch) ISR
redis
Ceph        PUSH primary-replica sync model with degrade mode 同步写入
Swift
Aurora      PUSH primary instance R(3)+W(4)>N(6) 
```
