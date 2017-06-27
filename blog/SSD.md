---
title: SSD
date: 2017-05-22 14:31:39
tags: storage
---

## Primer

Physical unit of flash memory
- Page
  unit for read & write
- Block
  unit for erase

物理特性
- Erase before re-write
- Sequential write within a block

![ssd](https://github.com/funkygao/blogassets/blob/master/img/ssd.png?raw=true)

## Optimal I/O for SSD

- I/O request size越好越好
- 要符号物理特性
  - page or block对齐
  - segmented sequential write within a block

http://codecapsule.com/2014/02/12/coding-for-ssds-part-6-a-summary-what-every-programmer-should-know-about-solid-state-drives/
http://www.open-open.com/lib/view/open1423106687217.html
