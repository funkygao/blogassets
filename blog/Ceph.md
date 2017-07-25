---
title: Ceph Internals
date: 2017-07-18 15:27:20
tags:
---

## RADOS

### Object

Object是最终落地存储文件的基本单位，可以类比fs block，默认4MB
如果一个文件 > object size，文件会被stripped，这个操作是client端完成的

It does not use a directory hierarchy or a tree structure for storage
It is stored in a flat-address space containing billions of objects without any complexity

Object组成
- key
- value
  在fs里用一个文件存储
- metadata
  可以存放在扩展的fs attr里
  因此对底层文件系统是有要求的，ext4fs对extended attr数量限制的太少，可以用xfs

```
  +---logical------+       +--physical layer -+
  |                |       |                  |
cluster -> pool -> pg ---> osd -> object -> fs files
                   |       |                 
                   +-crush-+
```

### Lookup

```
file = ino
obj = (ino, ono)
obj_hash = hash(ino, ono)
pg = obj_hash & (num_pg-1) // num_pg是2整数幂，例如codis里1024
                           // 官方推荐num_pg是osd总数的数百倍
                           // num_pg如果变化，会引起大量的数据迁移
osds_for_pg = crush(pg)    // list of osds，此处是唯一需要全局cluster map的地方
                           // 其他的，都是算出来的
primary = osds_for_pg[0]
replicas = osds_for_pg[1:]
```
![lookup](https://github.com/funkygao/blogassets/blob/master/img/ceph-lookup.png?raw=true)

### crush

目标
- 数据均匀的分布到集群中
- 需要考虑各个OSD权重的不同（根据读写性能的差异，磁盘的容量的大小差异等设置不同的权重）
- 当有OSD损坏需要数据迁移时，数据的迁移量尽可能的少

![crush](https://github.com/funkygao/blogassets/blob/master/img/ceph-crush.png?raw=true)

### PG

PG(placement group)与Couchbase里的vbucket一样，codis里也类似，都是presharding技术

在object_id与osd中间增加一层pg，减少由于osd数量变化造成大量的数据迁移
PG使得文件的定位问题，变成了通过crush定位PG的问题，数量大大减少
例如，1万个osd，100亿个文件，30万个pg: 100亿 vs 30万

线上尽量不要更改PG的数量，PG的数量的变更将导致整个集群动起来（各个OSD之间copy数据），大量数据均衡期间读写性能下降严重

```
Total PGs = (Total_number_of_OSD * 100) / max_replication_count
```

```
$ceph osd map pool1 object1
osdmap e566 pool 'pool1' (10) object 'object1' -> pg 10.bac4decb (10.3c) -> up [0,6,3] acting [0,6,3]
$
// osdmap e566: osd map epoch 566
// pool 'pool1' (10): pool name and pool id，pool id从0开始，每新建+1
// pg 10.bac4decb (10.3c): placement group number 10.3c(pg id是16进制，256个pg，那么他们的id: 0, 1, f, fe, ff)
                           pg id实际上是pool_id.pg_id(pool id=10, pg id=3c)
// up [0,6,3]: osd up set contains osd.0, osd.6, osd.3
```

### mon

相当于Ceph的zookeeper

mon quorum负责整个Ceph cluster中所有OSD状态(cluster map)，然后以增量、异步、lazy方式扩散到each OSD和client
mon被动接收osd的上报请求，作为reponse把cluster map返回，不主动push cluster map to osd
如果client和它要访问的PG内部各个OSD看到的cluster map不一致，则这几方会首先同步cluster map，然后即可正常访问

MON跟踪cluster状态：OSD, PG, CRUSH maps
同一个PG的osd除了向mon发送心跳外，还互相发心跳以及坚持pg数据replica是否正常

### Replication

是primary-replica model, 强一致性，读和写只能向primary发起请求
![read write](https://github.com/funkygao/blogassets/blob/master/img/ceph-rw.png?raw=true)
![read write](https://github.com/funkygao/blogassets/blob/master/img/ceph-rw1.jpg?raw=true)
其中replicas在复制到buffer时就ack，如果某个replica复制时失败，进入degrade状态

### scrub机制

read verify

## Questions

### Can Ceph replace facebook Haystack?

Haystack解决的实际上是metadata访问造成的IO问题，解决的方法是metadata完全内存，无IO
每个图片的read，需要NetApp进行3次IO: dir inode, file inode, file data
haystack每个图片read，需要1次IO，metadata保证在内存
在一个4TB的SATA盘存储20M个文件，每个inode如果需要500B，那么如果inode都存放内存，需要10GB内存
- 减少每个metadata的大小
  去掉大部分不需要的数据，例如uid, gid, atime等
  xfs里每个metadata占536字节，haystack每个图片需要10字节
- 减少metadata的总数量
  多个图片(needle)合并进一个大文件(superblock)

用户上传一张图片，facebook会将其生成4种尺寸的图片，每种存储3份，因此写入量是用户上传量的12倍。

Posix规范里的metadata很多对图片服务器不需要，
```
struct inode {
    loff_t              i_size;  // 文件大小
    struct list_head    i_list;  // backing dev IO list
    struct list_head    i_devices;
    blkcnt_t            i_blocks;
    struct super_block  *i_sb;
    struct timespec     i_atime;
    struct timespec     i_mtime;
    struct timespec     i_ctime;
    umode_t             i_mode;
    uid_t               i_uid;
    gid_t               i_gid;
    dev_t               i_rdev;
    unsigned int        i_nlink;
    ...
}
```

Ceph解决了一个文件到分布式系统里的定位问题(crush(pg))，但osd并没有解决local store的问题: 如果一个osd上存储过多文件(例如10M)，性能会下降明显

### Why EC is slow?

Google Colossus采用的是(6,3)EC，存储overhead=(6+3)/6=150%
把数据分成6个data block+3个parity block，恢复任意一个数据块需要6个I/O
生成parity block和恢复数据时，需要进行额外的encode/decode，造成性能损失

### Store small file, waste space?

object size是可以设置的

## References

http://ceph.com/papers/weil-crush-sc06.pdf
http://ceph.com/papers/weil-rados-pdsw07.pdf
http://ceph.com/papers/weil-ceph-osdi06.pdf
http://www.xsky.com/tec/ceph72hours/
