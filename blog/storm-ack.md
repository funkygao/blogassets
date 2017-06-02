---
title: storm acker
date: 2017-06-02 09:03:52
tags: PubSub
---

## Acker

对于Spout产生的每一个tuple, storm都会进行跟踪，利用RotatingMap存放内存，但有保护措施，不会打爆

当Spout触发fail动作时，storm不会自动重发失败的tuple，只是向Spout发送fail消息，触发Spout.fail回调，真正的重发需要在Spout.fail里实现

tuple tree，中间任意一个edge fail，会理解触发Spout的fail，但后面的Bolt的执行不受影响。做无用功？

![backtype.storm.daemon.acker](https://github.com/funkygao/blogassets/blob/master/img/acker.png?raw=true)

## Spout Executor

![spout executor](https://github.com/funkygao/blogassets/blob/master/img/spout.png?raw=true)

## Bolt Executor

![bolt executor](https://github.com/funkygao/blogassets/blob/master/img/bolt.png?raw=true)
