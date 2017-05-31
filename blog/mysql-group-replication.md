---
title: mysql group replication
date: 2017-05-11 16:50:06
tags: database
---

## 简介

GR是个mysql插件，通过原子广播协议、乐观事务冲突检测实现了高可用的多master集群
每个master都有全量数据，client side load balance write workload或者使用[ProxySQL](http://www.proxysql.com/)
读事务都是本地执行的
有2种模式
- 单主，自动选主
- 多主，active active master

与PXC是完全的竞争产品

## Requirements and Limitations

- InnoDB engine only, rollback uncommitted changes
- turn on binlog RBR
- GTID enabled
- each table MUST have a primary key或者not null unique key
- no concurrent DDL
- 至少3台master，至多9台，不需要slave
- auto_increment字段通过offset把各个master隔离开，避免冲突
- cascading foreign key not supported
- 只是校验write set，serializable isolation NOT supported
- 存在stale read问题，如果write/read不在一台member
- savepoints可能有问题

## Performance

http://mysqlhighavailability.com/an-overview-of-the-group-replication-performance/

80% throughput of a standalone MySQL server

## Internals

![GR](https://github.com/funkygao/blogassets/blob/master/img/gr-replication-diagram.png?raw=true)

### XCOM

eXtended COMmunications，一个Paxos系统

- 确保消息在所有member上相同顺序分发
- 动态成员，成员失效检测

理论基础 [Database State Machine](https://infoscience.epfl.ch/record/52305/files/IC_TECH_REPORT_199908.pdf)

事务的Update操作都在一个成员上执行，在Commit时把write-set以total order发送消息给每个成员；
每个成员上的certification进程检查事务冲突(first commit wins)，完成最终提交或回滚

Commit时的Paxos有2个作用
- certification，检测事务冲突
- propagate

Group Replication ensures that a transaction only commits after a majority of the members in a group have received it 
and agreed on the relative order between all transactions that were sent concurrently.

与multi-paxos不同，XCOM是multi-leader/multi-proposer：每个member都是leader of its own slots

### Certification

group_replication_group_name就是GTID里的UUID
GTID就是database version
mysql> select @@global.gtid_executed

transaction write set: [{updated_row_pk: GTID_EXECUTED}, {updated_row_pk: GTID_EXECUTED}, ...]

GTID是由certification模块负责的，由它来负责GTID GNO的increment
所有member会定期交换GTID_EXECUTED，所有member已经committed事务的交集：Stable Set.

### Transaction

![async apply](https://github.com/funkygao/blogassets/blob/master/img/gr-trans1.png?raw=true)
![async apply](https://github.com/funkygao/blogassets/blob/master/img/gr-trans2.png?raw=true)

### Distributed Recovery

向group增加新成员的过程: 获取missing data，同时cache正在发生的新事务，最后catch up

```
// 从现有member里通过mysql backup工具(mysqldump等)搞个backup instance

// phase 0: join
Joiner.join(), 通过total order broadcast发给每个member
生成view change binlog event: $viewID
group里每个member(包括Joiner)都会收到该view event
每个online member会把该binlog event排队到现有transaction queue里

// phase 1: row copy
Joiner pick a live member from the group as Donor // Donor可能会有lag
Doner transmits all data up to the joining moment: master/slave connection
for {
    if binlog.event.view_id == $viewID {
        Joiner.SQLThread.stop()
        break
    }

    if Doner.dies {
        reselect Donner
        goto restart
    }
}

// phase 2: catch up
joining moment后发生的binlogDonor发给Joiner，Joiner apply

catch up同步完成后，declare Joiner online，开始对外服务

// Joiner.leave()类似的过程
// crash过程会被detector发现，自动执行Joiner.leave()
```

这个过程与[mysql在线alter table设计](http://funkygao.github.io/2017/05/11/osc/)原理类似

#### binlog view change markers

group里member变化，会产生一种新的binlog event: view change log event.
view id就是一种logicl clock，在member变化时inrement
```
    +-----------------+
    | epoch | counter |
    +-----------------+
```
epoch在第一个加入group的member生成，作用是为了解决all members crash问题: avoid dup counter

### certification based replication

通过group communication和total order transaction实现synchronous replication

事务在单节点乐观运行，在commit时，通过广播和冲突检测实现全局数据一致性
它需要
- transactional database来rollback uncommitted changes
- primary keys to generate broadcast write-set
- atomic changes
- global ordering replication events

![certificationbasedreplication](https://github.com/funkygao/blogassets/blob/master/img/certificationbasedreplication.png?raw=true)


## Config

```
[mysqld]
log-bin
binlog-format=row
binlog-checksum=NONE
gtid-mode=ON
enforce-gtid-consistency
log-slave-updates
master-info-repository=TABLE
relay-log-info-repository=TABLE
transaction-write-set-extraction=MURMUR32

// GR
group_replication_group_name="da7bad5b-daed-da7a-ba44-da7aba5e7ab"
group_replication_local_address="host2:24901"
group_replication_group_seeds="host1:24901,host2:24901,host3:24901"
```

## FAQ

### GR是同步还是异步？

replication分为5步
```
master locally apply
master generate binlog event
master sending the event to slave(s)
slave IO thread add event to relay log
slave SQL thread apply the event from relay log
```

GR下，只有3是同步的: 把write set广播并得到majority certify confirm
广播时发送消息是同步的，但apply write set还是异步的:
```
member1: DELETE FROM a; // a table with many rows
member1: 产生一个非常大的binlog event

member1: group communicate the binlog event to all members(包括它自己)

其他member确认ok，那么member1就返回ok给client
client访问member1，那么数据是一致的

但其他member在异步apply binlog event，可能花很长时间，这时候client访问member2，可能不一致：
delete的数据仍然能读出来
```
![async apply](https://github.com/funkygao/blogassets/blob/master/img/cert-apply.png?raw=true)

## References

http://lefred.be/content/mysql-group-replication-about-ack-from-majority/
http://lefred.be/content/mysql-group-replication-synchronous-or-asynchronous-replication/
http://lefred.be/content/galera-replication-demystified-how-does-it-work/
http://www.tocker.ca/2014/12/30/an-easy-way-to-describe-mysqls-binary-log-group-commit.html
http://mysqlhighavailability.com/tag/mysql-group-replication/
http://mysqlhighavailability.com/mysql-group-replication-transaction-life-cycle-explained/
