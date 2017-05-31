---
title: asynchronous distributed snapshot
date: 2017-05-19 10:32:48
tags: algorithm
---

如何给分布式系统做个全局逻辑一致的快照?
Node State + Channel State

## 发送规则

```
node.recordState()
for conn in allConns {
    // before any conn's outbound msg
    conn.send(marker)
}
```

## 接收规则

```
msg = conn.recv()
if msg.isMarker() {
    t1 = now()
    if !node.stateRecorded() {
        node.recordState()
        Channel(conn) = []
    } else {
        Channel(conn) = msgsBetween(now(), t1)
        // in-flight msgs not applied on state
        node.state.apply(msgs before the marker)
    }
}
```

## Demo

![snapshot](https://github.com/funkygao/blogassets/blob/master/img/snapshot.jpg?raw=true)

```
a)
P为自己做快照P(red, green, blue)
在Channel(PQ)上 send(marker)

b)
P把绿球送给Q，这个消息是在marker后面
以此同时，Q把自己的橙色球送给P，此时Q(brown, pink)

c) 
Q在Channel(PQ)上收到marker // Q是接收者
Q为自己做快照Q(brown, pink)
Channel(PQ) = []

// 因为之前Q把自己的橙色球送给了P，因此Q也是发送者
在Channel(QP)上 send(marker)

d)
P收到橙色球，然后是marker
由于P已经记录了state, Channel(QP)=[orange, ]

最终的分布式系统的snapshot:
P(red, green, blue)
Channel(PQ) []
Q(brown, pink)
Channel(QP) = [orange, ]
```

## FAQ

### 如何发起

发起global distributed snapshot的节点，可以是一台，也可以多台并发

### 如何结束

所有节点上都完成了snapshot

### 用途

故障恢复

与Apache Storm的基于记录的ack不同，Apache Flink的failure recovery采用了改进的Chandy-Lamport算法
checkpoint coordinator是JobManager

data sources periodically inject markers into the data stream. 
```
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setParallelism(4)
env.enableCheckpointing(1000) // 数据源每1s发送marker(barrier)
```

Whenever an operator receives such a marker, it checkpoints its internal state. 
```
class StateMachineMapper extends FlatMapFunction[Event, Alert] with Checkpointed[mutable.HashMap[Int, State]] {
    private[this] val states = new mutable.HashMap[Int, State]()

    override def flatMap(t: Event, out: Collector[Alert]): Unit = {
        // get and remove the current state
        val state = states.remove(t.sourceAddress).getOrElse(InitialState)

        val nextState = state.transition(t.event)
        if (nextState == InvalidTransition) {
            // 报警
            out.collect(Alert(t.sourceAddress, state, t.event))
        } else if (!nextState.terminal) {
            // put back to states
            states.put(t.sourceAddress, nextState)
        }
    }

    override def snapshotState(checkpointId: Long, timestamp: Long): mutable.HashMap[Int, State] = {
        // barrier(marker) injected from data source and flows with the records as part of the data stream
        //
        // snapshotState()与flatMap()一定是串行执行的
        // 此时operator已经收到了barrier(marker)
        // 在本方法返回后，flink会自动把barrier发给我的output streams
        // 再然后，保存states(默认是JobManager内存，也可以HDFS)
        states
    }

    override def restoreState(state: mutable.HashMap[Int, State]): Unit = {
        // 出现故障后，flink会停止dataflow，然后重启operator(StateMachineMapper)
        states ++= state
    }
}
```

![snapshot](https://github.com/funkygao/blogassets/blob/master/img/checkpointing.png?raw=true)

## References

http://research.microsoft.com/en-us/um/people/lamport/pubs/chandy.pdf
https://arxiv.org/abs/1506.08603
https://ci.apache.org/projects/flink/flink-docs-master/internals/stream_checkpointing.html
https://github.com/StephanEwen/flink-demos/tree/master/streaming-state-machine
