---
title: kafka ext4
date: 2017-06-12 10:03:31
tags: PubSub
---

## big file

我目前使用ext4作为kafka存储，每个segment 1GB

Ext4针对大文件顺序访问的主要优化
- replace indirect blocks with extent(盘区)
  - ext3采用多层间接地址映射，操作大文件时产生很多随机访问
    - one extra block read(seek) every 1024 blocks(4MB)
    - ![ext2/3](https://github.com/funkygao/blogassets/blob/master/img/ext2-3.png?raw=true)
  - ext4 extent是一组连续的数据块
    ```
    +---------+--------+----------+
    | logical | length | physical |
    +---------+--------+----------+
    + 0       | 1000   | 200      |
    +---------+--------+----------+

    type ext4_extent struct {
      block    uint32 // first logical block extent covers
      len      uint16 // number of blocks covered by extent
      start_hi uint16 // high 16 bits of physical block
      start    uint32 // low 32 bits of physical block
    }
    ```

    ![ext4](https://github.com/funkygao/blogassets/blob/master/img/extentmap.png?raw=true)
- inode prealloc
  inode有很好的locality，同一目录下文件的inode尽量存放一起，加速了目录寻址性能
- Multiblock Allocator(MBAlloc)
  - ext3每次只能分配一个4KB(block size)的block
  - ext4支持一次性分配多个block
- 延迟分配
  defer block allocation to writeback time
- online defregmentation e4defrag
  ```
  allocate more contiguous blocks in a temporary inode

  read a data block from the original inode
  move the corresponding block number from the temporary inode to the original inode
  write out the page
  ```
  

## Benchmark

![benchmark](https://github.com/funkygao/blogassets/blob/master/img/fsbench.png?raw=true)

## Summary

![ext3 vs ext4](https://github.com/funkygao/blogassets/blob/master/img/ext3-vs-ext4.png?raw=true)

http://www.linux-mag.com/id/7271/
