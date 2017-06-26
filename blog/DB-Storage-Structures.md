---
title: DB Storage Structures
date: 2017-06-22 17:13:42
tags: storage
---

## KV

任何storage structure，数据都可以用k=>v来表示，不仅NoSQL，RDBMS也一样
例如InnoDB的primary key就是
```
primary_key => [column1, column2, ...]
```
secondary index也一样，因此在insert/update时有index maintaenance overhead，保持各个index的一致性

## B-Tree

Designed for optimal data retrieval performance, not data storage.

RDBMS, LMDB, MongoDB

![btree throughput](https://github.com/funkygao/blogassets/blob/master/img/btree.png?raw=true)

## LSM-Tree

每个SSTable的bloom filter只能帮助individual key lookup，对range query没用

## Fractal-Tree

与B-Tree类似，但通过buffer changes/data compression大大降低了disk random IO
