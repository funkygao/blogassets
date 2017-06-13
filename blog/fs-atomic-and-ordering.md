---
title: fs atomic and ordering
date: 2017-06-12 09:46:26
tags: architecture
---

应用层通过POSIX系统调用接口访问文件系统，但POSIX只规定了结果，底层的guarantee并没有提供，开发者只能去猜。
不同的文件系统的实现不同，那些不“可见”的部分，会造成很大的差异；同时，每个文件系统又有些配置，这些配置也造成理解的困难。

## Example

```
write(f1, "pp")
write(f2, "qq")
```

如果write不是原子的，那么可能造成文件大小变化了，但内容没变(State#A)
乱序造成State#C
![fs](https://github.com/funkygao/blogassets/blob/master/img/fs-crash.png?raw=true)

- size atomicity
- content atomicity
- calls out of order

## Facts

![persistence](https://github.com/funkygao/blogassets/blob/master/img/fs-persistence.png?raw=true)

### atomicity

- atomic single-sector overwrite
  目前绝大部分文件系统提供了atomic single-sector overwrite，主要是底层硬盘就已经提供了
- atomic append
  - 需要原子地修改inode+data block
  - ext3/ext4/reiserfs writeback不支持
- multi-block append
  目前绝大部分文件系统没有支持
- rename/link
  这类directory operation，基本上是原子的

### ordering

