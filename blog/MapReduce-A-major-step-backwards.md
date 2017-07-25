---
title: MapReduce - A major step backwards
date: 2017-07-17 14:19:27
tags: algorithm
---

## Author

都是数据库领域的权威

- Michael Stonebraker
  美国工程院院士，冯诺依曼奖的获得者，第一届SIGMOD Edgar F. Codd创新奖的得主，曾担任Informix CTO
  他在1992年提出对象关系数据库模型，在加州伯克利分校任计算机教授达25年，目前是麻省理工学院教授
  是SQL Server/Sysbase奠基人，87年左右，Sybase联合了微软，共同开发SQL Server。1994年(7y)，两家公司合作终止
  可以认为，Stonebraker教授是目前主流数据库的奠基人
- David J. DeWitt
  美国工程院院士
  In 1995, he was named a Fellow of the ACM and received the ACM SIGMOD Innovations Award 
  In 2009, ACM recognized the seminal contributions of his Gamma parallel database system project with the ACM Software Systems Award
  IEEE awarded him the 2009 IEEE Piore Award
  He was also a Technical Fellow at Microsoft, leading the Microsoft Jim Gray Systems Lab at Madison, Wisconsin.

## Background

2008年1月，一个数据库专栏读者让作者表达一下对MapReduce(OSDI 2004)的看法而引出

## Viewpoint

### MapReduce是历史后退

从IBM IMS的第一个数据库系统(1968)到现在的40年，数据库领域学到了3个经验，而MapReduce把这些经验完全摒弃，仿佛回到了60年代DBMS还没出现的时候
- schema is good
  防止garbage进入系统
  MapReduce由于涉及到(k, v)，一定也是可以用schema表达的
- 要把schema从应用中decouple
  存放在catalog里，而不是hard coding在应用层
- high level query is good
  70年代就充分讨论过应用通过SQL那样的语言访问DBMS好，还是更底层的语言
  MapReduce就像是DBMS的汇编语言

### MapReduce的实现糟糕

他们应该好好学学25年来并行数据库方面的知识

#### 1. Index vs Brute force

在非full scan场景下，index必不可少。此外，DBMS里还有query optimizer，而MapReduce里只能brute force full scan

MapReduce能提供自动的并行执行，但这些东西80年代就已经在学术界研究了，并在80年代末出现了类似Teradata那样的商业产品

#### 2. Record Skew导致性能问题

David J. DeWitt在并行数据库方向已经找到了解决方案，但MapReduce没有使用，好像skew问题根本不存在一样：
Map阶段，相同key的记录数量是不均衡的，这就导致Reduce阶段，有的reduce快、有的慢，最终Job的执行时间取决于最慢的那一个

#### 3. 吞吐率和IO问题

```
// map=1000 reduce=500
1000个map实例，每个map进程都会创建500个本地文件
reduce阶段，每个reduce都会到1000个map机器上PULL file
这会导致每个map机器上500个文件进行大量的random IO
```

而并行数据库，采用的是PUSH模型。而MapReduce要修改成PUSH模型是比较困难的，因为它的HA非常依赖它目前的PULL模型

### MapReduce并不新颖

他们以为他们发现了一个解决大数据处理的新模式，实际上20年前就有了

"Application of Hash to Data Base Machine and Its Architecture"/1983，是第一个提出把大数据集切分成小数据集的
Teradata使用MapReduce同样的技术卖他们的商业产品已经20多年了

就算MapReduce允许用户自己写map/reduce来控制，80年代中期的POSTGRES系统就支持用户自定义函数

### 缺少DBMS已经具备的功能

- Bulk loader
- Update
  MapReduce is read-only
- Transaction
- Views
  Hbase处已经提了
- Integrity constraints
  Schema处已经提了
- Referential integrity
  防止garbase进入系统

### 与DBMS工具不兼容

- BI tools
- Data mining tools
- Replication tools
- ER modelling

### SQL on hadoop

作者已经发行类似Pig那样的系统

### Comments

作者好像觉得MR是用来替代DBMS的

- MapReduce与DBMS的定位不同，它不是OLTP
  Index好，但cost也很高，在MR里index只会是成本而没有收益
  用正确的工具解决不同的问题
- Be a lover, not a fighter!
- Google本身已经证明了它的scalability
- With really large datasets and distributed sytems the RDBMS paradigms stop working
  and that is where a system like Mapreduce is needed
- 数据库那些人的问题：什么东西都是数据库
- 作者应该再写个“Airplanes: A major step backward”

## Jeff Dean在ACM of communication上面的回馈

