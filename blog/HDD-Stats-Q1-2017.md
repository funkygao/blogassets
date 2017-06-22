---
title: HDD Stats Q1 2017
date: 2017-06-14 11:14:03
tags: storage
---

![HDD Fail Rate](https://github.com/funkygao/blogassets/blob/master/img/blog-hd-stats-q1-2017-table.jpg?raw=true)

## Backblaze Storage Pod

把45块SATA盘存放到一台4U机器里，其中15块搞RAID6，ext4，可以空间是裸盘空间的87%，所有访问通过tomcat HTTPS
没有iSCSI，没有NFS，没有Fibre Channel
180TB成本$9,305，即每GB $0.0517

![storage pod](https://www.backblaze.com/blog/wp-content/uploads/2009/08/backblaze-storage-pod-main-components.jpg)

## Backblze Vaults cloud backup service

99.999999% annual durability
已经存储150PB，由1000个Storage Pod组成，40,000块盘，每天有10块盘损坏

每个Vault由20个Storage Pod组成，每个Pod有45块盘，即每个Vault有900块盘，一块盘如果是4TB，那么每个Vault可以存3.6PB
每个磁盘使用ext4文件系统，每个Vault有个7位数字id，例如555-1021，前3位代表data center id，后4位是vault id
有个类似name service的服务，client备份前先request name service获取vault id，之后client直接与相应的vault进行backup IO(https)
![pod](https://www.backblaze.com/blog/wp-content/uploads/2015/03/blog-vault-tome-1.jpg)

每个文件被分成20 shards = 17 data shars + 3 parity shards，存放在ext4
每个shard有checksum，如果损坏，可以从其他17个shards恢复
如果某个Vault有1个pod crash了，backup write的parity会变成2，如果3个pod坏了，那么也可以写，但parity就不存在了，如果此时再坏一个pod，数据无法恢复了

## References

https://www.backblaze.com/blog/vault-cloud-storage-architecture/
https://www.backblaze.com/blog/hard-drive-failure-rates-q1-2017/
