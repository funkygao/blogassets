---
title: OtterTune
date: 2017-12-11 09:28:11
tags: algorithm
---

## DB tuning问题

传统方法：DBA把生产库copy到另外一个机器，回放sql，根据metrics尝试调节某一个参数看效果，再尝试多个

- 数据库的调优参数过多，通常都是数百个，而且新的版本会增加更多参数
- 同一个db的参数之间是互相依赖的
- 依赖于运行环境
  例如，增加innobuffer，在某些条件下是正回馈，但如果物理机内存开始swap，增加它会起副作用
- 跟业务数据有关
 
## OtterTune解决办法

通过controller收集数据库的参数和负载metrics以及hardware profile，定期向中央报告。
分析系统从中央取得原始数据，进行分析，提供参数配置建议，生效后，持续分析比较，找出合适的参数配置

### target workload

- latency
- throughput

### Controller

就是agent,定期把通过配置文件配置的数据库实例的paramters & metrics通过HTTP POST以JSON格式上传
同时，它是trial and error的执行者，对参数不断的尝试取得样本数据，它必须有修改数据库参数权限，甚至restart db

#### PostgreSQL

- SELECT version()
- parameters
  SHOW ALL
- metrics
  - select * from pg_stat_archiver/pg_stat_bgwriter/pg_stat_database/pg_statio_user_indexes/...

### 分析

- 通过factor analysis(FA)对收集来的metrics进行降维
  例如，read_in_bytes/read_in_kbytes
- 通过k-means对metrics进行聚类
- 通过Lasso线性回归，来发现哪些参数对target workload有重大影响
- workload mapping
  通过Euclidean计算目标workload与历史采样数据的相似度，找出最相似的
- 推荐配置
  通过GaussianProcess(GP)回归进行训练
- controller接受推荐的配置apply on DB，并收集新的数据

## References

http://db.cs.cmu.edu/papers/2017/p1009-van-aken.pdf
https://github.com/cmu-db/ottertune
