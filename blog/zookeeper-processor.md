---
title: zookeeper processor
date: 2017-05-25 08:39:35
tags: 一致性
---

Chain of Responsibility
为了实现各种服务器的代码结构的高度统一，不同角色的server对应不同的processor chain

```
interface RequestProcessor {
    void processRequest(Request request) throws RequestProcessorException;
}
```

LeaderZooKeeperServer.java
![Leader](https://github.com/funkygao/blogassets/blob/master/img/zkprocessor2.png?raw=true)
![Leader](https://github.com/funkygao/blogassets/blob/master/img/zkleader.jpeg?raw=true)

FollowerZooKeeperServer.java
![Follower](https://github.com/funkygao/blogassets/blob/master/img/zkprocessor3.png?raw=true)
![Follower](https://github.com/funkygao/blogassets/blob/master/img/zkfollower.jpeg?raw=true)

ZooKeeperServer.java
```
func processPacket() {
    submitRequest()
}

func submitRequest(req) {
    firstProcessor.processRequest(req)
}
```
