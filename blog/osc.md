---
title: mysql在线alter table设计
date: 2017-05-11 15:42:46
tags: database
---

## 主要逻辑

```
// 合法性检查
// 包括：是否有外键、用户权限、表是否合法、是否有唯一键等

// 创建变更记录表
CREATE /* changelog table */ TABLE _tbs_c

// 创建影子表
CREATE /* shadow table */ TABLE _tbl_s LIKE tbl

// 在影子表上应用alter语句
ALTER TABLE _tbl_s STATEMENT

// 开始行拷贝线程 tbl -> _tbl_s
// 开始binlog接收和应用线程 binlog -> _tbl_s

// 等待行拷贝线程完成
// 通知binlog线程收工
// 等待binlog线程结束

// 开始切换
LOCK TABLES tbl WRITE
RENAME TABLE tbl TO _tbl_old, _tbl_s TO tbl 
UNLOCK TABLES
```

### 确定行拷贝chunk范围

```
select id from 
   (select id from a where id>=0 and id<=3001 order by id asc limit 1000) select_osc_chunk 
order by id desc limit 1;
```

### 行拷贝in chunk

```
begin;
insert ignore into `a`.`_a_gho` (`id`, `value`)
  (select `id`, `value` from `a`.`a` force index (`PRIMARY`)
      where (((`id` > ?) or ((`id` = ?))) and ((`id` < ?) or ((`id` = ?)))) 
          lock in share mode
  )
commit;
```

## 关键点

### async binlog worker如何判断所有数据变更已经完成

binlog worker向changelog table发一行记录，在收到这个记录时，即表示完成

### RENAME race condition with DML

mysql内部保证，LOCK TABLE后，如果有DML与RENAME并发操作，那么在UNLOCK TABLES时，RENAME
一定获取最高优先级，即：RENAME一定会先执行。
否则，会丢数据:
```
LOCK TABLE WRITE
INSERT // blocked
RENAME // blocked

UNLOCK TABLE
INERT // 如果INSERT先执行，那么它会插入原表
RENAME // 原表被rename to tbl_old，刚才INSERT的数据丢失: 存放在了tbl_old
```

### LOCK, RENAME

如果在一个mysql连接内执行LOCK; RENAME，那么会失败
解决办法：创建2个mysql连接，分别执行LOCK和RENAME
