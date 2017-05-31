---
title: etcd3 vs zookeeper
date: 2017-05-25 17:07:34
tags: 一致性
---

## etcd v3独有的特性

- get and watch by prefix, by interval
- lease based TTL for key sets
- 获取历史版本数据(这个非常有用)
- watcher功能丰富
  - 支持index参数，不会lose event
  - streaming
  - recursive

## zk独有的特性

- ephemeral znode

