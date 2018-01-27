---
title: BigData related
date: 2018-01-18 08:44:54
tags: architecture
---

## Open Source

- Storage
  - HDFS
    - RCFile
    - ORCFile
    - Parquet
  - Kudu
- NoSQL
  - Cassandra/DynamoDB
  - Hbase
  - Voldemort
  - Espresso
- SQL-on-hadoop
  - Hive(on MR)
    SQL解析，物理执行是通过生成map/reduce job完成的
  - ad-hoc query to group,filter,aggregate data
    - Presto
      Facebook开发的Hive，但后来放弃了，转而开发了Presto。
      一个Presto query可以跨越多个数据源
    - Impala(Dremel的启发下开发的C++)
      Cloudera虽然支持Hive，但自己开发了Impala。底层HDFS存储Parquet，建议用Kudu替换HDFS。
      Kudu是columnar datastore，但不提供SQL解析执行，SQL部分由Impala完成。
      定位于短查询，如果节点失效，查询会重头开始，没有fault-tolerant。
    - SparkSQL(Shark)
      Shark最初是在Hive的代码基础上进行的，后来重新实现了：SparkSQL
    - Stinger/Tez
    - Hive-on-Spark/Hive-on-Tez
      Hortonworks
    - Google Dremel
      只能查询一个table，不能join
      - Apache Drill
      - Druid
      - IBM BigSQL
- Stream Processing
  - Storm
  - Heron
  - Spark mini batch
  - Flink
  - Samza
- ETL
- Network OS
  - yarn
  - mesos
  - k8s
  - Apache Tez
    一个Hadoop DAG技术框架，解决MR的问题一个替代方案，依赖YARN
- tools
  - sqoop

## 序列化

- thrift
- avro
- hessian
- protocol buffer

## Apache Kylin

MOLAP engine built on Hive

### OLAP

Slice/Dice/Drill-down/Roll-up/Pivot

- ROLAP
- MOLAP
- HOLAP

### Cube

Cube   = all combination of dimensions
Cuboid = one combination of dimensions(all cuboids)

Cube is immutable!

- One fact table
  has ever growing records
- A few dimension tables
  relatively static, like users and products
- Hive tables must be synced into Kylin first
- Measures
  - sum
  - count
  - distinct count(HyperLogLog)
  - avg
  - max
  - min
- Incremental Build
  - Segment
    - 与ES的实现思路类似
      - query时aggregate
      - merge small cubes into a larger one
    - cube is immutable
  - 只支持按时间维度

### Visualization

- saiku
- Caravel
- Zeppelin
- superset
- Cboard
