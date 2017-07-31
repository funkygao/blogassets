---
title: RCFile, Parquet, Dremel
date: 2017-07-13 10:12:54
tags: storage
---

## Overview

hadoop的存储格式：TextFile, SequenceFile, Hive优化的RCFile, 类似google dremel的Parquet

Data placement for MapReduce
- row store
- column store
- hybrid PAX store

## RCFile - Record Columnar File

一种PAX的实现：Table先水平切分成多个row group，每个row group再垂直切分到不同的column
这样保证一个row的数据都位于一个block(避免一个row要读取多个block)，列与列的数据在磁盘上是连续的存储块

table = RCFile -> block -> row group -> (sync marker, metadata header(RLE), column store(gzip))

![rcfile](https://github.com/funkygao/blogassets/blob/master/img/rcfile.png?raw=true)

For a table, all row groups have the same size.

ql/src/java/org/apache/hadoop/hive/ql/io/RCFile.java

### Usage

```
CREATE TABLE demo_t (
  datatime string,
  section string,
  domain string,
  province string,
  city string,
  idc string)
STORED AS RCFILE;
```

## ORCFile

## Parquet

## References

http://web.cse.ohio-state.edu/hpcs/WWW/HTML/publications/papers/TR-11-4.pdf
http://www.ece.eng.wayne.edu/~sjiang/ECE7650-winter-14/Topic3-RCFile.pdf
