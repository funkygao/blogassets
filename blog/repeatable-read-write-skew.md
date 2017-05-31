---
title: mysql repeatable read write skew
date: 2017-05-12 09:18:24
tags: database
---

## Isolation

教科书里的4种isolation
- read uncommitted: 即dirty read，可能读到其他rollback的数据
- read committed: 即non-repeatable read，同一个txn内读一条数据多次，结果可能不同
- repeatable read: 一个txn内读一条数据多次结果相同，但不保证读多个数据的时候也相同(phantom read)
- serialized

但它只是从锁的实现来描述的，不适用于MVCC。
各个数据库产品，虽然采用了这些isolation名字，但语义各不相同，很多与教科书里的定义不符

### MySQL

不会出现phantom read。
MySQL里的RR其实是Snapshot Isolation，只有Serialized是完全基于锁

### PostgreSQL

实际上只有2个隔离级别：Read Committed和Serialized
而Serialized是基于MVCC的无锁实现，即Serialized Snapshot

## MVCC Snapshot

存在write skew问题

甲在一个银行有两张信用卡，分别是A和B。银行给这两张卡总的信用额度是2000，即A透支的额度和B透支的额度相加必须不大于2000：A+B<=2000。

A账号扣款
```
begin;
a = select credit from a
b = select credit from b
if (a + b) + amount <= 2000 {
    update a set credit = credit + amount
}
commit
```

B账号扣款
```
begin;
a = select credit from a
b = select credit from b
if (a + b) + amount <= 2000 {
    update b set credit = credit + amount
}
commit
```

假设现在credit(a)=1000, credit(b)=500, 1500<=2000
甲同时用a账号消费400，b账号消费300
在mysql RR下，2个事务都成功，但2个事务结束后
credit(a)=1400, credit(b)=700, 2100>2000

如果是serialized隔离级别，则没有问题：一个事务会失败

在mysql RR下，可以通过应用层加约束来避免write skew

## 结论for mysql

不能期望加了一个事务就万事大吉，而要了解每种隔离级别的语义。
- 涉及单行数据事务的话，只要 Read Committed ＋ 乐观锁就足够保证不丢写
- 涉及多行数据的事务的话，Serializable 隔离环境的话不会出错，但是你不会开
- 如果开 Repeatable Read （Snapshot）隔离级别，那么可能会因为 Write Skew 而丢掉写

如果是金融业务，尽量不要用MySQL

