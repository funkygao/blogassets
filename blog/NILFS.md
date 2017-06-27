---
title: NILFS
date: 2017-06-26 18:49:36
tags: misc
---

## Intro

New Implementation of a Log-Structured File System，included in Linux 2.6.30 kernel

- take snapshot非常简单，只要记录一下version就可以了
- 尤其在随机的小文件读写效率更高
- 在SSD上，NILFS2具有绝对性能优势

```
insmod nilfs2.ko 
mkfs – t nilfs2 /dev/sda8 
mount – t nilfs2 /nilfs /dev/sda8
```

## Benchmark

![small file](https://github.com/funkygao/blogassets/blob/master/img/smallfile.png?raw=true)
![large file](https://github.com/funkygao/blogassets/blob/master/img/largefile.png?raw=true)

## vs Journal File System

JFS保存在日志里的只有metadata，而LFS利用日志记录一切

## References

http://www.linux-mag.com/id/7345/
