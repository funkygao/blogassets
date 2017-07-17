---
title: InnoDB MVCC
date: 2017-05-16 08:17:17
tags: database
---

## Basic

InnoDB中通过Undo log实现了txn rollback和MVCC，而并发控制(isolation)通过锁来实现
Undo log分为insert undo和update undo(delete是一种特殊的update)，回滚时
- insert只要把insert undo log丢弃即可
- update需要通过DB_ROLL_PTR DB_TRX_ID找到事务修改前的版本并恢复

与redo log不同的是，磁盘上不存在单独的undo log文件，所有的undo log均存放在主ibd数据文件中（表空间），即使客户端设置了每表一个数据文件也是如此

## 内部存储

InnoDB为每行row都reserved了隐藏字段(system column)
- DB_ROW_ID
- DB_TRX_ID
- DB_ROLL_PTR

```
typedef ib_uint64_t ib_id_t;

typedef ib_id_t row_id_t;
typedef ib_id_t trx_id_t;
typedef ib_id_t roll_ptr_t;
```

![syscol](https://github.com/funkygao/blogassets/blob/master/img/undolog.jpg?raw=true)

## Undo log实现方式

- 事务以排他锁的形式修改原始数据
- 把修改前的数据存放于undo log，通过回滚指针与主数据关联
- 修改成功（commit）啥都不做，失败则恢复undo log中的数据（rollback）

## Demo

![demo](https://github.com/funkygao/blogassets/blob/master/img/undo1.jpg?raw=true)
![demo](https://github.com/funkygao/blogassets/blob/master/img/undo2.jpg?raw=true)
![demo](https://github.com/funkygao/blogassets/blob/master/img/undo3.jpg?raw=true)

Innodb中存在purge线程，它会查询那些比现在最老的活动事务还早的undo log，并删除它们

## Issues

### 回滚成本

当事务正常提交时Innbod只需要更改事务状态为COMMIT即可，不需做其他额外的工作
而Rollback如果事务影响的行非常多，回滚则可能成本很高

### write skew

InnoDB通过Undo log实现的MVCC在修改单行记录是没有问题的，但多行时就可能出问题
```
begin;
update table set col1=2 where id=1; // 成功，创建了undo log
update table set col2=3 where id=2; // 失败
rollback;
```
回滚row(id=1)时，由于它没有被lock，此时可能已经被另外一个txn给修改了，这个回滚会破坏已经commit的事务

如果要解决这个问题，需要应用层进行控制

另外一个例子
```
国家规定一个家庭最多养3只宠物(constraint)，Alice和Bob是一家人，他们现在有dog(1)+cat(1)
如果并发地，Alice再买一只猫，而Bob再买一只狗，这2个事务就会write skew
因为repeatable read：一个事务提交之后才visible by other txn
TxnAlice      TxnBob
cat=cat+1     dog=dog+1
这2个事务都成功了，但却破坏了constraint
```

## 锁

RR隔离级别下，普通select不加锁，使用MVCC进行一致性读取，即snapshot read
update, insert, delete, select ... for update, select ... lock in share mode都会进行加锁，并且读取的是当前版本: READ COMMITTED读
除了lock in share mode是S锁，其他都是X锁

## References

https://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html
https://blog.jcole.us/innodb/
http://jimgray.azurewebsites.net/WICS_99_TP/
