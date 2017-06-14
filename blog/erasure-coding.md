---
title:  Reed-Solomon erasure code
date: 2017-06-14 09:20:19
tags: [storage, algorithm]
---

## Intro

纠错码在RAID、备份、冷数据、历史数据存储方面使用广泛
也有人利用它把一份数据分散到多个cloud provider(e,g. S3,Azure,Rackspace)，消除某个供应商的依赖: cloud of cloud

## Usage

一份文件，大小x，分成n个数据块+k个校验块，能容忍任意k个数据块或者校验块错误，即至少要n个块是有效的
```
$encode -data 4 -par 2 README.md # n=4 k=2
Opening README.md
File split into 6 data+parity shards with 358 bytes/shard.
Writing to README.md.0
Writing to README.md.1
Writing to README.md.2
Writing to README.md.3
Writing to README.md.4 # README.md.4 and README.md5 are parity shards
Writing to README.md.5

$rm -f README.md.4 README.md.5 # remove the 2 parity shards

$decode -out x README.md
Opening README.md.0
Opening README.md.1
Opening README.md.2
Opening README.md.3
Opening README.md.4
Error reading file open README.md.4: no such file or directory
Opening README.md.5
Error reading file open README.md.5: no such file or directory
Verification failed. Reconstructing data
Writing data to x

$diff x README.md
$ # all the same
```

### Prod Use Case

一个文件，被等分成17个shards+3个校验块，共计20块，容忍任意3块错误，而成本只增加了3/17=17%

## Algorithm

文件内容‘ABCDEFGHIJKLMNOP’，4+2
给定n和k，encoding矩阵是不变的

![coding 4=>4+2](https://github.com/funkygao/blogassets/blob/master/img/ec.png?raw=true)
![deocde after lost 2 shards](https://github.com/funkygao/blogassets/blob/master/img/ec1.png?raw=true)

## References

http://pages.cs.wisc.edu/~yadi/papers/yadi-infocom2013-paper.pdf
