---
title: hybrid logical clock
date: 2017-05-19 07:51:28
tags: algorithm
---

分布式事务，为了性能，目前通常提供SI/SSI级别的isolation，通过乐观冲突检测
而非2PC悲观方式实现，这就要求实现事务的causality，通常都是拿逻辑时钟实现total order
例如vector clock就是一种，zab里的zxid也是；google percolator里的total order算是
另外一种逻辑时钟，但这种方法由于有明显瓶颈，也增加了一次消息传递

但逻辑时钟无法反应物理时钟，因此有人提出了混合时钟，wall time + logical time，分别是
给人看和给机器看，原理比较简单，就是在交互消息时，接收方一定sender event happens before receiver

但wall time本身比较脆弱，例如一个集群，有台机器ntp出现问题，管理员调整时间的时候出现人为
错误，本来应该是2017-09-09 10:00:00，结果typo成2071-09-09 10:00:00，后果是它会传染给集群
内所有机器，hlc里的wall time都会变成2071年，人工无法修复，除非允许丢弃历史数据，只有等
到2071年那一天系统会自动恢复，wall time部分也就失去了意义

要解决这个问题，可以加入epoch
```
HLC
+-------+-----------+--------------+
| epoch | wall time | logical time |
+-------+-----------+--------------+
```

修复2071问题时，只需把epoch+1

