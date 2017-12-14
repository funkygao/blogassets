---
title: OtterTune
date: 2017-12-11 09:28:11
tags: algorithm
---

## Controller

就是agent,定期把通过配置文件配置的数据库实例的paramters & metrics通过HTTP POST以JSON格式上传

### PostgreSQL

- SELECT version()
- parameters
  SHOW ALL
- metrics
  - select * from pg_stat_archiver/pg_stat_bgwriter/pg_stat_database/pg_statio_user_indexes/...

##

## References

http://db.cs.cmu.edu/papers/2017/p1009-van-aken.pdf
https://github.com/cmu-db/ottertune
