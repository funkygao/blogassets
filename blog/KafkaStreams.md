---
title: Kafka Streams
date: 2017-06-22 15:24:10
tags: PubSub
---

Kafka 0.10提供，可以部分替代SparkStreaming/Storm/Samza/Flink，好处是仅仅依赖kafka，全部通过SDK实现流式处理

通过KeyValueStore interface实现stateful processor，目前有2个实现
- in memory
- RocksDB

其实跟[dbus](https://github.com/funkygao/dbus)的设计差不多

```
JsonDeserializer<Purchase> purchaseJsonDeserializer = new JsonDeserializer<>(Purchase.class);
JsonSerializer<Purchase> purchaseJsonSerializer = new JsonSerializer<>();
JsonSerializer<RewardAccumulator> rewardAccumulatorJsonSerializer = new JsonSerializer<>();
JsonSerializer<PurchasePattern> purchasePatternJsonSerializer = new JsonSerializer<>();
StringDeserializer stringDeserializer = new StringDeserializer();
StringSerializer stringSerializer = new StringSerializer();

TopologyBuilder topologyBuilder = new TopologyBuilder();
topologyBuilder.addSource("SOURCE", stringDeserializer, purchaseJsonDeserializer, "src-topic")
    .addProcessor("PROCESS", CreditCardAnonymizer::new, "SOURCE")
    .addProcessor("PROCESS2", PurchasePatterns::new, "PROCESS")
    .addProcessor("PROCESS3", CustomerRewards::new, "PROCESS")

    // kafka(src-topic) -> SOURCE -> PROCESS -+-> PROCESS2 
    //                                        +-> PROCESS3

    .addSink("SINK", "patterns", stringSerializer, purchasePatternJsonSerializer, "PROCESS2")
    .addSink("SINK2", "rewards", stringSerializer, rewardAccumulatorJsonSerializer, "PROCESS3")
    .addSink("SINK3", "purchases", stringSerializer, purchaseJsonSerializer, "PROCESS");

KafkaStreams streaming = new KafkaStreams(topologyBuilder, streamingConfig);
streaming.start();
```

## References

https://cwiki.apache.org/confluence/display/KAFKA/KIP-28+-+Add+a+processor+client
