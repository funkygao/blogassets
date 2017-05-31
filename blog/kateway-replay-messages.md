---
title: kateway replay messages
date: 2017-05-17 07:52:34
tags: PubSub
---

## Issue

consumer有需求回放/快进消息，目前[kateway](https://github.com/funkygao/gafka/tree/master/cmd/kateway)具有该功能：
用户在web console上把offset设置到指定位置

但由于机器里[kateway](https://github.com/funkygao/gafka/tree/master/cmd/kateway)正在消费源源不断的消息，checkpoint会overwrite这个指定的offset
这就要求用户先关闭消费进程，然后web console上操作，再启动消费进程: not user friendly
在不影响性能前提下，对其进行改进

## Solution

```
_, stat, err := cg.kz.conn.Get(path)
if cg.lastVer == -1 {
    // 第一次commit offset
    cg.lastVer = stat.Version
} else if cg.lastVer != stat.Version {
    // user manually reset the offset checkpoint
    return ErrRestartConsumerGroup
}

// 也可能在Get后，用户恰好操作了“回放”，通过CAS解决这个问题

switch err {
case zk.ErrNoNode:
    return cg.kz.create(path, data, false)

case nil: 
    newStat, err := cg.kz.conn.Set(path, data, stat.Version)
    if err != nil {
        cg.lastVer = newStat.Version
    }
    return err

default:
    return err
}
```
