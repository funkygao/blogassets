---
title: bigdata story
date: 2017-07-14 08:06:16
tags: cloud
---

大数据，主要是因为Google的Google File System(2003)， MapReduce(OSDI 2004)，BigTable(2006)三篇论文
GFS论文是最经典的，后面google的很多论文都遮遮掩掩的
Chubby(2006)
Dremal(2010)
Borg(2015)

微软在bing项目上的投入，使得微软掌握了大规模data center的建设管理能力，掌握了A/B testing平台，学会了大规模数据分析能力，其中一些成为了Azure的基础，它使得微软顺利从一家软件公司过渡到了云计算公司。
微软内部支撑大数据分析的平台Cosmos是狠狠的抄袭了Google的File system却很大程度上摒弃了MapReduce这个框架
所谓MapReduce的意思是任何的事情只要都严格遵循Map Shuffle Reduce三个阶段就好。其中Shuffle是系统自己提供的而Map和Reduce则用户需要写代码。Map是一个per record的操作。任何两个record之间都相互独立。Reduce是个per key的操作，相同key的所有record都在一起被同时操作，不同的key在不同的group下面，可以独立运行
Map是分类(categorize)操作，Reduce是aggregate

To draw an analogy to SQL, map is like the group-by clause of an aggregate query. Reduce is analogous to the aggregate function (e.g., average) that is computed over all the rows with the same group-by attribute.

Flume Java本质上来说还是MapReduce，但是它采取了一种delayed execution的方式，把所有相关的操作先存成一个execution DAG,然后一口气执行。
这和C#里的LINQ差不多。这种方式就产生了很多optimization的机会了。具体来说大的有两个:
- 一个是怎么样把若干个Map或者Reduce整成一个
- 一个是怎么样把一系列的operation整成一个MapReduce job

Hadoop三大批发商分别是Cloudera，Hortonworks以及MapR。
MapR(印度人CTO是Google GFS项目组，后用C++写了自己的HDFS)、Cloudera(源于BerkeleyDB卖给Oracle)都成立于2009年，Hortonworks(源于Yahoo Hadoop分拆) 2011，目前Cloudera一家独大
Hortonworks基本上就是Yahoo里的Hadoop团队减去被Cloudera挖走的Doug Cutting, Hadoop的创始人。这个团队的人做了不少东西，最初的HDFS和Hadoop MapReduce, ZooKeeper,以及Pig Latin。
MapR的印度人CTO，目前已经跳槽到了Uber

Hortonworks本来是一个发明了Pig的公司。Pig是它们的亲儿子。现在他们作为一个新成立的公司，不像Cloudera或者MapR那样另起炉灶，却决定投入HIVE的怀抱，要把干儿子给捧起来
Pig发表在SIGMOD2008，之后Hive也出来了
时至今日，我们必须说Pig是被大部分人放弃了：原来做Pig的跑去了Hortonworks，改行做Hive了
HIVE已经事实上成为了Hadoop平台上SQL和类SQL的标杆和事实的标准。

2008年Hadoop的一次会议，facebook的2个印度人讲了他们hackathon的项目Hive，他们说SQL的SELECT FROM应该是FROM SELECT
2010年Hive论文发表，直到2014年那2个印度人一直在Hive PMC，后来出来创业大数据公司
后来，做Pig的人看上了Hive，Hortonworks把自己人塞进PMC，现在那2个印度人已经除名了，Hive基本上是Hortonworks和Cloudera的天下，Cloudera做企业需要的安全方面东西，Hortonworks做性能提升
于是，Pig的人成了Hive主力，做Hive的人跑光了

Dynamo: A flawed architecture
http://jsensarma.com/blog/?p=55

Hbase开始的时候是一个叫Powerset的公司，这个公司是做自然语言搜索的。公司为了能够实现高效率的数据处理，做了HBase。2008年的时候这个公司被卖给了微软

VLDB(Very Large Data Base), 世界数据库业界三大会议之一；另外两个是ICDE和sigmod，另外还有KDD。
在数据库领域里面，通常SIGMOD和VLDB算是公认第一档次，ICDE则是给牛人接纳那些被SIGMOD VLDB抛弃的论文的收容所，勉强1.5吧，而且有日渐没落的趋势。
CIDR在数据库研究领域是个奇葩的会议，不能说好不能说不好。因为是图灵机得主Michael Stonebraker开的，给大家提供个自娱自乐的场所。所以很多牛人买账，常常也有不错的论文。但是更多的感觉是未完成的作品。

Dremal出来没多久，开源社区，尤其是Hadoop的批发商们都纷纷雀跃而起。Cloudera开始做Impala，MapR则做了Drill，Hortonworks说我们干脆直接improve HIVE好了，所以就有了包括Apache Tez在内的effort
Dremal的存储是需要做一次ETL这样的工作，把其他的数据通过转化成为它自己的column store的格式之后才能高效率的查询的。

Dremal出来后，Cloudera 2012年做出了Impala，MapR搞了个Drill，Hortonworks也许最忽悠也许最实际，说我们只需要改善 Hive就好

Spark是迄今为止由学校主导的最为成功的开源大数据项目
UCBerkeley作为一个传统上非常有名的系统学校，它曾经出过的系统都有一个非常明确的特点，可用性非常高
