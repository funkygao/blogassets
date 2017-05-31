---
title: how DBMS works
date: 2017-05-16 16:31:29
tags: database
---

![dbms](https://github.com/funkygao/blogassets/blob/master/img/datastructuredb1.png?raw=true)

### Undo log

- Oracle和MySQL机制类似
- MS SQL Server里，称为transaction log
- PostgreSQL里没有undo log，它通过mvcc系统表实现，每一行存储多个版本

### Redo log

- Oracle和MySQL机制类似
- MS SQL Server里，称为transaction log
- PostgreSQL里称为WAL
