---
title: TimescaleDB
date: 2017-09-28 08:42:41
tags: database
---

## time series数据的现有方案问题

- 传统RDBMS
  - 无法支持high ingest rate，尤其index无法完全放入内存后
  - delete的成本高
    时序数据是有retention的
- NoSQL和time series database
  Cassandar, MongoDB
  OpenTSDB, InfluxDB
  - 通常缺乏丰富的查询接口，复杂查询是高延时
- Hadoop/Spark
  ingest rate可以高，但查询慢

## TimescaleDB解决办法

### Intro

PostgreSQL上的一个插件，目前只支持单机部署，cluster功能还在开发
因此，PQ具备的功能它都有，此外还针对tsdb提供了一些方便的函数

- hypertable
  相当于多个chunk(物理)上的逻辑统一
- chunk
  相当于shard，物理属性，通过time interval和[partition key]进行路由
  chunk就是PQ的table

### 解决方法

#### high inject rate

传统mysql/postgresql，受限于index，如果无法放到内存，性能会大大降低

在100亿条数据的hypertable，单机单磁盘，仍然可以享受10万插入/秒(in batch).

TimescaleDB solves this through its heavy utilization of time-space partitioning, even when running on a single machine. 
So all writes to recent time intervals are only to tables that remain in memory, and updating any secondary indexes is also fast as a result.

#### retention

删除数据时，不是delete by row，而是delete by chunk，把整个chunk(table)删除就快了

    SELECT drop_chunks(interval '7 days', 'conditions');

#### chunk partition

目前不支持adaptive time intervals，需要在创建hypertable时手工指定chunk_time_interval(默认1个月)
