---
title: aliware
date: 2017-06-01 15:34:25
tags: cloud
---

https://www.aliyun.com/aliware

- EDAS
  - Enterprise Distributed Application Service
  - RPC framework + elasticjob + qconf + GTS + Dapper + autoscale
  - 鉴权数据下放到服务机器，避免性能瓶颈
- MQ
- DRDS
  - TDDL proxy
  - avg() => sum()/count()
  - 扩容，切换时client可能会失败，需要client retry
  - 通过/* TDDL: XXX*/ sql注释来实现特定SQL路由规则
  - 可以冷热数据分离
- ARMS
  streaming processing，具有nifi式visualized workflow editor
- GTS
  ```
  @GtsTransaction(timeout=1000*60)
  public void transfer(DataSource db1, DataSource db2) {
      // 强一致，但涉及的事务比较大时，性能下降非常明显
      // 通过soft state MQ的最终一致性吞吐量要好的多
      db1.getConnection().execute("sql1")
      db2.getConnection().execute("sql2")
  }
  ```
- SchedulerX
- CSB
  cloud service bus，相当于api gateway，包含协议转换
