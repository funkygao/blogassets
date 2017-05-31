---
title: 两段锁 2PL
date: 2017-05-11 15:33:41
tags: database
---

事务开始后就处于加锁阶段，一直到执行ROLLBACK和COMMIT之前都是加锁阶段。ROLLBACK和COMMIT使事务进入解锁阶段

事务遵守两段锁协议是可串行化调度的充分条件，而不是必要条件

MS SQL Server默认采用2PL实现isolation，Oralce/PostgreSQL/MySQL InnoDB默认使用MVCC

MySQL 2PL提供2种锁
- shared(read)
- exclusive(write)

读写互斥，但读读不互斥

MySQL serialized isolation
![2pl](https://github.com/funkygao/blogassets/blob/master/img/phantomread2plmysql1.png?raw=true)
