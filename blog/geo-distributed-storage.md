---
title: geo-distributed storage system
date: 2017-06-27 18:22:58
tags: storage
---

## Primer 

Network of network.

### Why

- scale 
  one datacenter too small
- disaster tolerance
  protect data against catastrophes 
- availability 
  keep working after intermittent problems access 
- locality
  serve users from a nearby datacenter

### Challanges

- high latency 
- low bandwidth 
- congestion 
  overprovisioning prohibitive 
- network partitions 

## Replication

```
        read-write    |  state machine
----------------------+-----------------
sync    ABD           |  Paxos
async   logical clock |  CRDT

ABD(Alike Backup Delegate)
```

## Congestion

### Movivation

- bandwidth across datacenter is expensive 
- storage usage subject to spikes

如何准备带宽？
- 按峰值
  成本高，利用率低
- 按均值
  网络拥塞

### Solution

- weak consistency
  async replication
- priority messages
  should be small

### vivace algorithm

read-write algorithm based on ABD

算法
- 把数据和timestamp分离
  把数据和元数据分离，元数据优先级更高
- replicate data locally first
- replicate timestamp remotely with prioritized msg
- replicate data remotely in background

write
```
obtain timestamp ts
write data,ts to f+1 temporary local replicas (big msg)
write only ts to f+1 real replicas (small msg, cross datacenter) 
in background, send data to real replicas (big msg)
```

read
```
read ts from f+1 replicas (small msg)
read data associated with ts from 1 replica (big msg, often local) 
write only ts to f+1 real replicas (small msg, cross datacenter)
```

## Transaction

### snapshot isolation(SI)

total ordering of update transactions

问题
- it orders the commit time of all transactions
  even those that do not conflict with each other
- forbids some scenarios we want to allow for efficiency

### parallel snapshot isolation(PSI)

causality: if T1 commits at site S before T2 starts at site S then T2 does not commit before T1 at any site

```
SI                  PSI
------------------- ---
1 commit time       1 commit time per site
1 timeline          1 timeline per site
read from snapshot  read from snapshot at site
no w-w conflict     no w-w conflict
                    causality property
```

#### idea1: preferred sites

每个key分配一个唯一的preferred site(例如，在长沙的用户它的preferred site就是长沙IDC)，在该site上，可以使用fast commit(without cross-site communication)
类似primary/backup，不同的是preferred sites不是强制的，key可以在任何site修改

但问题是
- what if many sites often modify a key?
- no good way to assign a preferred site to key

#### idea2: CRDT

### serializable

SI/PSI都有write skew问题
通过transaction chains[sosp 2013]可以有效地提供serializable isolation

## References

http://dprg.cs.uiuc.edu/docs/vivace-atc/vivace-atc12.pdf
