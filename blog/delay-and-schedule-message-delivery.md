---
title: delay and schedule message delivery
date: 2017-05-15 08:11:24
tags: PubSub
---

## 使用场景

- 业务需要
- 通过它可以实现XA的prepare/commit/rollback，从而实现与其他系统的原子提交

## 实现

### [kateway](https://github.com/funkygao/gafka/tree/master/cmd/kateway)

通过mysql作为WAL，并通过background worker(actor)来实现调度/commit/rollback

### 优先队列

以message due time作为优先级进行存储，配合worker
message rollback可以通过发送一个tombstone message实现
但由于worker的async，无法在rollback时判断是否真正rollback成功：
一条消息要5分钟后发送，在5分钟到达时，client可能恰好要取消，这时候，rollback与worker
之间存在race condition，需要正常处理这个一致性：
要么，取消失败，消息被发出
要么，取消成功，消息不发出
不能，取消成功，消息被发出
```
// worker
for {
    if msg := peek(queue); msg.due() {
        msg = pop(queue)
        if msg.isTombstone() {
            // the msg is cancelled
        } else {
            publish(msg)
        }
    }
}
```
