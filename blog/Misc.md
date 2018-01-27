---
title: 知名公司功能模块的实现笔记
date: 2017-12-14 10:06:48
tags: [algorithm, architecture]
---

## Storage

### 微信支付的交易记录

之前kv，每个用户一个key(相当于redis list)，这样问题是：
- value会大
- 无法根据条件filter value

改进后：
没有用户多个value，其中1个root value，保存metadata，其他value为data
多value解决了以前单value大的问题，但：
- 多了一次请求
  先root，再value
- root成为新瓶颈
  - 可以把root也变成一个link list
    按照时间倒排，新的是head，老的是tail
  - 但越以前的数据，越慢

## 算法

### biparties graph(二部图/二分图)

对多种查询条件进行归并缓存，提高缓存命中率

查询条件：

```
    (row1, row5) => (field3, field7)
    (row2) => (field2, field4)
    (row3) => (field3, field5)
    (row4, row6) => (field1, field4, field8)
```

利用二部图把图切分，把零散查询归并为少数集中的查询

## M$ Cloud Design Pattern

- cache-aside
- ciruit breaker
- compensating transation
- competing consumers
- CQRS
- event sourcing
- external configuration store
- federated identity
- gatekeeper(gateway)
- health endpoint monitoring
- index table
- lead election
- materialized view
- pipes and filters
- priority queue
- queue-based load leveling
- retry
- runtime reconfiguration
- scheduler agent supervisor
- sharding
- static content hosting
- throttling
- valet key
