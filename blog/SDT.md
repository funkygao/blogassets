---
title: Swinging Door Trending
date: 2017-06-08 14:52:01
tags: algorithm
---

SDT旋转门流式压缩，有损算法，对于变化频率不高的时间序列数据，压缩比很高，但可能丢到峰值、波谷数据
通过一条由起点和终点确定的直线代替一系列连续数据点
通过线性差值“还原”数据，pref sample <--插值--> next sample

![用a点到e点之间的直线代替数据点(a,b,c,d,e)](https://github.com/digoal/blog/raw/master/201608/20160813_01_pic_003.png)

vs facebook gorilla
http://www.vldb.org/pvldb/vol8/p1816-teller.pdf
