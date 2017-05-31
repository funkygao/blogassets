---
title: materialized view
date: 2017-05-22 10:05:02
tags: database
---

物化试图，可以理解为cache of query results, derived result
觉得用“异构表”可能更贴切

![materialized view](https://github.com/funkygao/blogassets/blob/master/img/materializedview.png?raw=true)

与试图不同，它是物理存在的，并由数据库来确保与主库的一致性
它是随时可以rebuilt from source store，应用是从来不会更新它的: readonly

MySQL没有提供该功能，但通过[dbus](https://github.com/funkygao/dbus)可以方便构造materialized view
PostgreSQL提供了materialized view

## References

https://docs.microsoft.com/en-us/azure/architecture/patterns/materialized-view
