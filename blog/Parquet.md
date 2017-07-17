---
title: Parquet (TODO)
date: 2017-07-13 10:12:54
tags: storage
---

hadoop的存储格式：TextFile, SequenceFile, Hive优化的RCFile, 类似google dramel的Parquet

Data placement for MapReduce
- row store
- column store
- hybrid PAX store

logical view -> storage -> physical view

## RCFile - Record Columnar File

是一种PAX的实现：Table先水平切分成多个row group，每个row group再垂直切分到不同的column
这样保证一个row的数据都位于一个block(避免一个row要读取多个block)，列与列的数据在磁盘上是连续的存储块

RCFile是一种“允许按行查询，提供了列存储的压缩效率”的混合列存储格式。它的

解决嵌套的schema结构化存储问题，跟google dramel类似
在行式存储中一行的多列是连续的写在一起的，在列式存储中数据按列分开存储

核心思想是使用“record shredding and assembly algorithm”来表示复杂的嵌套数据类型

Parquet格式的存储中，一个schema的树结构有几个叶子节点，实际的存储中就会有多少column

