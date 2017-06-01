---
title: KafkaSpout
date: 2017-06-01 18:04:04
tags: PubSub
---

## config

```
fetchSizeBytes  = 1024 * 1024 * 2 // 1048576=1M by default FetchRequest
fetchMaxWait = 10000              // by default
forceFromStart = false

Config.TOPOLOGY_ACKERS                   // if 0, acking disabled
Config.TOPOLOGY-ACKER-EXECUTORS   = 20
Config.TOPOLOGY_MAX_SPOUT_PENDING = 5000 // max in-flight tuples per spout
Config.TOPOLOGY_MESSAGE_TIMEOUT_SECS     // 30s by default
```

## emit/ack/fail flow

```
class PartitionManager {
    SortedSet<Long> _pending, failed = new TreeSet();
    LinkedList<MessageAndRealOffset> _waitingToEmit = new LinkedList();

    func EmitState next(SpoutOutputCollector collector) {
        // 从LinkedList _waitingToEmit里取一条消息
        MessageAndRealOffset toEmit = (MessageAndRealOffset)this._waitingToEmit.pollFirst();
        // emit时指定了messageID
        // BasicBoltExecutor.execute会通过template method自动执行_collector.getOutputter().ack(input)
        // 即KafkaSpout.ack -> PartitionManager.ack
        collector.emit(tup, new PartitionManager.KafkaMessageId(this._partition, toEmit.offset));
    }

    func ack(Long offset) {
        this._pending.remove(offset)
    }

    func fail(Long offset) {
        this.failed.add(offset);
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

