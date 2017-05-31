---
title: oklog
date: 2017-05-22 11:22:58
tags: PubSub
---

injecter负责write优化(WAL)，让storage node负责read优化
与[RocketMQ](https://funkygao.github.io/2017/05/17/RocketMQ/)类似: CQRS
- injecter = commit log
- storage node = consume log

不同在于：storage node是通过pull mode replication机制实现，可以与injecter位于不同机器
而[RocketMQ](https://funkygao.github.io/2017/05/17/RocketMQ/)的commit log与consume log是在同一台broker上的

- kafka couples R/W，无法独立scale
- CQRS decouples R/W，可以独立scale

## produce

Producer通过forwarder连接到多个injecter上，injecter间通过gossip来负载均衡，load高的会通过与forwarder协商进行redirect distribution

## query

scatter-gather
