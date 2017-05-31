---
title: PostgreSQL MVCC
date: 2017-05-16 11:54:13
tags: database
---

## Internals

MySQL通过undo log记录uncommitted changes，与此不同，PostgreSQL store all row versions in table data structure.

每个row有2个隐藏字段
- Tmin insert时的trx id
- Tmax delete时的trx id

## INSERT

![insert](https://github.com/funkygao/blogassets/blob/master/img/mvcc_insert.png?raw=true)

## DELETE

![delete](https://github.com/funkygao/blogassets/blob/master/img/mvcc_delete.png?raw=true)

DELETE操作并不会马上物理删除，而是VACUUM进程调度进行purge

## UPDATE

![update](https://github.com/funkygao/blogassets/blob/master/img/mvcc_update.png?raw=true)

