---
title: LSM for SSD
date: 2017-05-16 09:33:59
tags: storage
---

## Basics

![leveldb](https://github.com/funkygao/blogassets/blob/master/img/leveldb.png?raw=true)

- (immutable)memtable: sorted skiplist
- SSTable: sorted string table
- L0里的SSTable们，key可能会overlap，因为他们是直接从immutable memtable刷出来的

### SSTable file

```
+-------------------+-------------------------+------------+
| index block(16KB) | bloom-filter block(4KB) | data block |
+-------------------+-------------------------+------------+
```

### Get(key)

```
locate key in memtable
if found then return
locate key in immutable memtable
if found then return
for level:=0; level<=6; level++ {
    // 对于level0，需要遍历所有SSTable，因为keys overlap
    // 但leveldb为了解决这个问题，除了Bloomfilter，也限制了L0的文件数量，一旦超过8，就compact(L0->L1)
    // 对于其他level，由于已经sorted，可以直接定位SSTable
    //
    // 最坏情况下，Get(key)需要读8个L0以及L1-L6，共14个文件
    locate key in SSTable in $level 
    if found then return
}
```

## SSD

主流SSD，例如[Samsung 960 Pro](http://www.anandtech.com/show/10754/samsung-960-pro-ssd-review)，可以提供440K/s random read
with block size=4KB

LSM是为传统硬盘设计的，在SSD下，可以做优化

## 优化

LSM-Tree的主要成本都在compaction(merge sort)，造成IO放大(50倍)
- 读很多文件到内存
- 排序
- 再写回到磁盘

![io](https://github.com/funkygao/blogassets/blob/master/img/ioamplification.png?raw=true)

要优化compaction，可以把LSM Tree变小，RocksDB是通过压缩实现的

在SSD下，可以考虑把key和value分离，在LSM Tree里只保存sorted key和pointer(value)，value直接保存在WAL里

key: 16B
pointer(value): 16B

2M个k/v，需要64MB
2B个k/v，需要64GB

## Reference

https://www.usenix.org/system/files/conference/fast16/fast16-papers-lu.pdf
