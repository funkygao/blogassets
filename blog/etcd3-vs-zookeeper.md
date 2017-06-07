---
title: etcd3 vs zookeeper
date: 2017-05-25 17:07:34
tags: 一致性
---

## etcd v3独有的特性

- get and watch by prefix, by interval
- lease based TTL for key sets
- runtime reconfiguration
- point in time backup
- extensive metrics
- 获取历史版本数据(这个非常有用)
  multi-version
- mini transation DSL
  ```
  Tx.If(
      Compare(Value("foo"), ">", "bar"),
      Compare(Value(Version("foo"), "=", 2),
      ...
  ).Then(
      Put("ok", "true")...
  ).Else(
      Put("ok", "false")...
  ).Commit()
  ```
- leases
  ```
  l = CreateLeases(15*second)
  Put(foo, bar, l)
  l.KeepAlive()
  l.Revoke()
  ```
- watcher功能丰富
  - streaming watch
  - 支持index参数，不会lose event
  - recursive
- off-heap
  内存中只保留index，大部分数据通过mmap映射到boltdb file
- incremental snapshot

## zk独有的特性

- ephemeral znode
- non-blocking full fuzzy snapshot
  Too busy to snap, skipping
- key支持在N Millions
- on-heap

## etcd2

- etcd2的key支持在10K量级，etcd3支持1M量级
  - 原因在于snapshot成本，可能导致0 qps，甚至reelection

## comparison

### memory footprint

2M 256B keys
```
etcd2 10GB
zk    2.4GB
etcd3 0.8GB
```

## References

https://coreos.com/blog/performance-of-etcd.html
