---
title: KafkaSpout
date: 2017-06-01 18:04:04
tags: PubSub
---

## config

### Topology config

- TOPOLOGY_WORKERS
  - 整个topology在所有节点上的java进程总数
  - 例如，设置成25，parallelism=150，那么每个worker进程会创建150/25=6个线程执行task
- TOPOLOGY_ACKER_EXECUTORS = 20
  - 不设或者为null，it=TOPOLOGY_WORKERS，即one acker task per worker
  - 设置为0，表示turn off ack/reliability
- TOPOLOGY_MAX_SPOUT_PENDING = 5000 
  ```
  (defn executor-max-spout-pending [storm-conf num-tasks]
    (let [p (storm-conf TOPOLOGY-MAX-SPOUT-PENDING)]
      (if p (* p num-tasks))))
  ```
  - max in-flight(not ack or fail) spout tuples on a single spout task at once
  - 如果不指定，默认是1

- TOPOLOGY_BACKPRESSURE_ENABLE = false
- TOPOLOGY_MESSAGE_TIMEOUT_SECS     
  30s by default

### KafkaSpout config

```
fetchSizeBytes  = 1024 * 1024 * 2 // 1048576=1M by default FetchRequest
fetchMaxWait    = 10000           // by default
forceFromStart  = false
```

## emit/ack/fail flow

```
class PartitionManager {
    SortedSet<Long> _pending, failed = new TreeSet();
    LinkedList<MessageAndRealOffset> _waitingToEmit = new LinkedList();

    func EmitState next(SpoutOutputCollector collector) {
        if (this._waitingToEmit.isEmpty()) {
            // 如果内存里数据都发出，就调用kafka consumer一次性批量填充内存_waitingToEmit
            // 填充时，如果发现failed里有东西，那么就从head of failed(offset) FetchRequest: 重发机制
            this.fill();
        }

        // 从LinkedList _waitingToEmit里取一条消息
        MessageAndRealOffset toEmit = (MessageAndRealOffset)this._waitingToEmit.pollFirst();
        // emit时指定了messageID
        // BasicBoltExecutor.execute会通过template method自动执行_collector.getOutputter().ack(input)
        // 即KafkaSpout.ack -> PartitionManager.ack
        collector.emit(tup, new PartitionManager.KafkaMessageId(this._partition, toEmit.offset));

        // 该tuple处于pending state
    }

    // Note: a tuple will be acked or failed by the exact same Spout task that created it

    func ack(Long offset) {
        this._pending.remove(offset)
    }

    func fail(Long offset) {
        this.failed.add(offset);
        // kafka consumer会reset offset to the failed msg，重新消费
    }
}

class TopologyBuilder {
    public BoltDeclarer setBolt(String id, IBasicBolt bolt, Number parallelism_hint) {
        return setBolt(id, new BasicBoltExecutor(bolt), parallelism_hint);
    }
}

class BasicBoltExecutor {
    public void execute(Tuple input) {
         _collector.setContext(input);
         try {
             _bolt.execute(input, _collector);
             _collector.getOutputter().ack(input);
         } catch(FailedException e) {
             _collector.getOutputter().fail(input);
         }
    }
}
```

### Bolt ack

KafkaSpout产生的每个tuple，Bolt必须进行ack，否则30s后KafkaSpout会认为emitted tuple tree not fully processed，进行重发
```
class MyBolt {
    public void execute(Tuple tuple) {
        _collector.emit(new Values(foo, bar))
        _collector.ack(tuple)
    }
}
```

### OOM

如果消息处理一直不ack，累计的unacked msg越来越多，会不会OOM?
NO
KafkaSpout只保留offset，不会保存每条emitted but no ack/fail msg

## spout throttle

[1.0.0](http://storm.apache.org/2016/04/12/storm100-released.html)之前，只能用TOPOLOGY_MAX_SPOUT_PENDING控制
但这个参数很难控制，它有一些与其他参数配合使用才能生效的机制，而且如果使用Trident语义又完全不同
[1.0.0](http://storm.apache.org/2016/04/12/storm100-released.html)之后，可以通过backpressure
![backpressure](https://github.com/funkygao/blogassets/blob/master/img/backpressure.png?raw=true)

## Storm messaging

- intra-worker
  Disruptor
- inter-worker
  0MQ/Netty

![storm messaing](https://github.com/funkygao/blogassets/blob/master/img/storm-internal-message-queues.png?raw=true)

## References

http://www.michael-noll.com/blog/2013/06/21/understanding-storm-internal-message-buffers/
http://jobs.one2team.com/apache-storms/
http://woodding2008.iteye.com/blog/2335673
