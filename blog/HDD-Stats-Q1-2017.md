---
title: HDD Stats Q1 2017
date: 2017-06-14 11:14:03
tags: storage
---

生产环境下游8万个磁盘，其中1800个是启动盘，即1800台服务器，每台服务器挂40个盘
每个磁盘3~8TB

![HDD Fail Rate](https://github.com/funkygao/blogassets/blob/master/img/blog-hd-stats-q1-2017-table.jpg?raw=true)

## Backblaze Storage Pod

把45块SATA盘存放到一台4U机器里，其中15块搞RAID6，ext4，可以空间是裸盘空间的87%，所有访问通过tomcat HTTPS
没有iSCSI，没有NFS，没有Fibre Channel
180TB成本$9,305，即每GB $0.0517

![storage pod](https://www.backblaze.com/blog/wp-content/uploads/2009/08/backblaze-storage-pod-main-components.jpg)

## References

https://www.backblaze.com/blog/hard-drive-failure-rates-q1-2017/
