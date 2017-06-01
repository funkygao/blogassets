---
title: how DBMS works
date: 2017-05-16 16:31:29
tags: database
---

## Basics

![dbms](https://github.com/funkygao/blogassets/blob/master/img/datastructuredb1.png?raw=true)

### Undo log

- Oracle和MySQL机制类似
- MS SQL Server里，称为transaction log
- PostgreSQL里没有undo log，它通过mvcc系统表实现，每一行存储多个版本

### Redo log

- Oracle和MySQL机制类似
- MS SQL Server里，称为transaction log
- PostgreSQL里称为WAL

## Query Optimization

大部分是基于Selinger的论文，动态规划算法，把这个问题拆解成3个子问题
- cost estimation
  以I/O和CPU的成本衡量
- relational equivalences that define a search space
- cost-based search

## Concurrency Control

Gray论文
- 区分细粒度和粗粒度的锁
  数据库是个分层结构 hierarchical structure
- 提出了多种隔离级别
  最初都是2PL实现的serializable isolation

## Database Recovery

IBM的ARIES算法(1992)，Algorithm for Recovery and Isolation Exploiting Semantics
ARIES can only update the data in-place after the log reaches storage
确保在恢复时，已经commit的事务要redo，未commit的事务要undo
redo log是物理的，undo log是逻辑的

No Force, Steal
- database need not write dirty pages to disk at commit time
  由于有redo log，update pages are written to disk lazily after commit
  No Force
- database can flush dirty pages to disk at any time
  由于有undo log，uncommitted(dirty) pages can be written to disk by the buffer manager
  Steal

ARIES为each page保存LSN，disk page是数据管理和恢复的基本单位，page write是原子的

ARIES crash recovery分成3步
- analysis phase
  从前向后，determine winners & losers
- redo phase
  如果是Force(在commit前刷dirty pages)，就不需要redo stage了
  repeat history
- undo phase
  从后向前，undo losers

ARIES数据结构
- xaction table
- dirty page table
- checkpoint

Example
```
After a crash, we find the following log:
0   BEGIN CHECKPOINT
5   END CHECKPOINT (EMPTY XACT TABLE AND DPT)
10  T1: UPDATE P1 (OLD: YYY NEW: ZZZ)
15  T1: UPDATE P2 (OLD: WWW NEW: XXX)
20  T2: UPDATE P3 (OLD: UUU NEW: VVV)
25  T1: COMMIT
30  T2: UPDATE P1 (OLD: ZZZ NEW: TTT)

Analysis phase:
Scan forward through the log starting at LSN 0.
LSN 5: Initialize XACT table and DPT to empty.
LSN 10: Add (T1, LSN 10) to XACT table. Add (P1, LSN 10) to DPT.
LSN 15: Set LastLSN=15 for T1 in XACT table. Add (P2, LSN 15) to DPT.
LSN 20: Add (T2, LSN 20) to XACT table. Add (P3, LSN 20) to DPT.
LSN 25: Change T1 status to "Commit" in XACT table
LSN 30: Set LastLSN=30 for T2 in XACT table.

Redo phase:
Scan forward through the log starting at LSN 10.
LSN 10: Read page P1, check PageLSN stored in the page. If PageLSN<10, redo LSN 10 (set value to ZZZ) and set the page's PageLSN=10.
LSN 15: Read page P2, check PageLSN stored in the page. If PageLSN<15, redo LSN 15 (set value to XXX) and set the page's PageLSN=15.
LSN 20: Read page P3, check PageLSN stored in the page. If PageLSN<20, redo LSN 20 (set value to VVV) and set the page's PageLSN=20.
LSN 30: Read page P1 if it has been flushed, check PageLSN stored in the page. It will be 10. Redo LSN 30 (set value to TTT) and set the page's PageLSN=30.

Undo phase:
T2 must be undone. Put LSN 30 in ToUndo.
Write Abort record to log for T2
LSN 30: Undo LSN 30 - write a CLR for P1 with "set P1=ZZZ" and undonextLSN=20. Write ZZZ into P1. Put LSN 20 in ToUndo.
LSN 20: Undo LSN 20 - write a CLR for P3 with "set P3=UUU" and undonextLSN=NULL. Write UUU into P3.
```

ARIES是为传统硬盘设计的，顺序写，但成本也明显：修改1B，需要redo 1B+undo 1B+page 1B=3B
what if in-place update with SSD?

## 分布式

mid-1970s 2PC 一票否决

## References

https://blog.acolyer.org/2016/01/08/aries/
http://cseweb.ucsd.edu/~swanson/papers/SOSP2013-MARS.pdf
https://www.cs.berkeley.edu/~brewer/cs262/Aries.pdf
