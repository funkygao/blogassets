---
title: gnatsd
date: 2017-05-31 15:33:01
tags: PubSub
---

2011年用Ruby完成了一个版本，后来用golang重写，目前只维护golang版本
NATS = Not Another Tibco Server

协议类似redis
持久化通过上层的NATS Streaming完成

竞品是0mq/nanomsg/aeron

https://github.com/real-logic/Aeron
