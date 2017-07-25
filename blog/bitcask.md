---
title: bitcask
date: 2017-07-13 08:02:10
tags: storage
---

A Log-Structured Hash Table for KV storage
Riak默认采用的存储引擎就是bitcask，在开发bitcask的时候，LevelDB还没有开放出来
刚听到的消息，Riak的公司basho，已经离解散不远了，CTO已经离职，公司好像只有1、2人了

## Data

![data files](https://github.com/funkygao/blogassets/blob/master/img/bitcask1.jpg?raw=true)

older data files will be merged and compacted

![data entry](https://github.com/funkygao/blogassets/blob/master/img/bitcask2.jpg?raw=true)

## Index

reside in mem

![hash index](https://github.com/funkygao/blogassets/blob/master/img/bitcask3.jpg?raw=true)

读取file_id对应文件的value_pos开始的value_sz个字节，就得到了我们需要的value值

### Write

```
append to active data file
update hash table index in memory
```

## Hint File

为了冷启动时加快index生成速度：重建hash table时，不需要full scan data files，只scan much smaller hint file

![hint file](https://github.com/funkygao/blogassets/blob/master/img/bitcask4.jpg?raw=true)

## Issues

- 由于index是包含所有key的，因此可以容纳的数据量跟内存有很大关系，每个index至少占40B
- 不支持range/sort操作
  如果把hash table变成skiplist，而merge sort older data files，就可以支持range了
