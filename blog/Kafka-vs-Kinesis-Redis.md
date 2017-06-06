---
title: Kafka vs Kinesis vs Redis
date: 2017-06-05 11:15:08
tags: PubSub
---

## vs Kinesis

US East region，要支持10万/秒的吞吐量，Kinesis需要的费用是4787美元/月

https://www.quora.com/Amazon-Kinesis-versus-Apache-Kafka-which-of-them-is-the-most-proven-and-high-performance-oriented

## vs Redis PubSub

- redis OOM
- kafka replay messages, horizontal scale, HA, async Pub
