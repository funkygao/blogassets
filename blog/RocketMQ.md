---
title: RocketMQ解读
date: 2017-05-17 10:38:32
tags: PubSub
---

## Features

- Producer Group
  发送事务消息时，作为TC，要多机，保存事务状态表{offset: P/C/R}
- Broker tag-based message filter 
- 定时消息，不支持任意精度，只是特定level: 5s, 10s, 1m等 queueID=delayLevel-1
  因此，应该不支持message revoke
- 区分commit log和consume log，有点类似WAL和table关系
  可以把它们放在不同FS下，但没有更细粒度的
  增加了一个分发步骤的好处：可以不分发

## Commit Log

```
${rocketmq.home}\store\commitlog\${fileName}

fileName[n] = fileName[n-1] + mappedFileSize
为了保证mappedFileSize相同，在每个file tail加padding，默认1GB
```

每条消息
```
QueueOffset针对普通消息，存的是consume log里的offset；如果事务消息，是事务状态表的offset
+---------+-------+-----+---------+------+-------------+----------------+----------------+
| MsgSize | Magic | CRC | QueueID | Flag | QueueOffset | PhysicalOffset | SysFlag(P/C/R) |
+---------+-------+-----+---------+------+-------------+----------------+----------------+

+--------------+------------------+-----------+---------------+----+------+-------+------+
| ProducedTime | ProduderHostPort | StoreTime | StoreHostPort | .. | Body | Topic | Prop |
+--------------+------------------+-----------+---------------+----+------+-------+------+
```

每次append commit log，会同步调用dispatch分发到consume queue和索引服务
```
new DispatchRequest(topic, queueId, 
    result.getWroteOffset(), result.getWroteBytes(),
    tagsCode, msg.getStoreTimestamp(), 
    result.getLogicsOffset(), msg.getKeys(),
    // Transaction
    msg.getSysFlag(),
    msg.getPreparedTransactionOffset());
```

### queue

仅仅是逻辑概念，可以通过它来参与producer balance，类似一致哈希里的虚拟节点
每台broker上的commitlog被本机所有的queue共享，不做任何区分

```
broker1: queue0, queue2
broker2: queue0, 

then, topicA has 3 queues:
broker1_queue0, broker1_queue2, broker2_queue0

producer.selectOneMessageQueue("topicA", "broker1", "queue0")
```

消息的局部顺序由producer client保证

### Question

- 如何实现retention by topic: 没有实现
  仅仅根据commit log file的mtime来判断是否过期，虽然里面混杂多topics
- 如何I/O balancing
- 如何压缩
- 如果CRC出错，那么所有topic都受影响?
- 为什么要存StoreHostPort？如何迁移topic：无法迁移
- 写commit log需要加锁，这个锁粒度太大，相当于db level lock，而非table level
- broker的脑裂问题
- failover
- topic的commit log是分散在所有broker上的

## Consume Queue

```
${rocketmq.home}/store/consumequeue/${topicName}/${queueId}/${fileName}
```

读一条消息，先读consume queue(类似mysql的secondary index)，再读commit log(clustered index)

没有采用sendfile，而是通过mmap：因为random read

```
+---------------------+-----------------+------------------------+
| CommitLogOffset(8B) | MessageSize(4B) | MessageTagHashcode(8B) |
+---------------------+-----------------+------------------------+
```

虽然消费时，consume queue是顺序的，但接下来的commit log几乎都是random read，此外
如何优化压缩？光靠pagecache+readahead是远远不够的

## Producer

```
TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic()); // from local cache or name server
MessageQueue mq = topicPublishInfo.selectOneMessageQueue(lastBrokerName);
sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, timeout);
```

## Transaction

```
// 2PC，2 messages

// Phase1
producer group write redolog
producer group send a message(type=TransactionPreparedType) to broker
broker append it to CommitLog and return MessageId
broker will not append it to consume queue

// Phase2
producer group write redolog
producer group send a message(type=TransactionCommitType, msgId=$msgId) to broker
broker find the message with msgId in CommitLog and clone it and append it to CommitLog(type=TransactionCommitType|TransactionRollbackType)
if type == TransactionCommitType {
    broker append commit log offset to consume queue
}
```

### State Table

保存在broker，默认1m扫一次

```
24B, mmap

+-----------------+------+-----------+-----------------------+--------------+
| CommitLogOffset | Size | Timestamp | ProducerGroupHashcode | State(P/C/R) |
+-----------------+------+-----------+-----------------------+--------------+

prepare消息，insert table
commit/rollback消息，update table
```

对于未决事务，根据随机向Producer Group里的一台发请求CHECK_TRANSACTION_STATE
Producer Group根据redolog(mmap)定位状态
Producer Group信息存放在namesvr

### Problems

- Producer不再是普通的client，它已经变成server(TC)，而且要求不能随便shutdown
- Producer Group里写redolog的机器死了怎么办
