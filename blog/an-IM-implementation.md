---
title: an IM implementation
date: 2017-06-29 13:38:03
tags: PubSub
---

## Architecture

Pros
- 通过kafka将上行、下行消息分开处理
- 通道和逻辑分离

Cons
- 逻辑服务器对session服务器(router)的感知，通过一致性哈希解决扩容问题
- 虽然可以进行多IDC部署，但设计上并没有充分考虑
  目前腾讯其实是所有IDC router 全局同步的，这样各个IDC甚至可以直连本IDC的router模块查询
- 还没有实现离线消息
- 实现上对可靠性的处理比较naive，存在多处丢失消息的case
- 配置上静态服务地址绑定
  - 虽然可以通过VIP LB
  - 每台comet需要配置所有logic addrs
  - 每台logic需要配置所有router addrs
  - 每台job需要配置所有comet addrs

### 系统组成

- 接入服务器/前置机(comet)
  负责消息的接收和投递
  client只接触到它，通过websocket/http
- 业务服务器(logic)
  认证、路由查询、消息kafka持久化
- session存储服务器(router)
  保存路由信息表
- push服务器(job)
  从kafka取消息，通过router里的信息找到对应comet服务器，进行投递

### kafka

```
type KafkaMsg struct {
    OP       string  
    RoomId   int32  
    ServerId int32 
    SubKeys  []string 
    Msg      []byte  
}
```

## Q & A

### What if comet crash?

Client端是有token的，session不会有问题

### What if router crash?

big trouble

### What if job crash?

job is stateless，会产生kafka consumer group rebalance，不影响。
如果只有一个job，重启即可

### A发消息给B,C,D，产生几条kafka msg?

多条(在push的时候，job->comet，这里可以合并一些RPC请求)

但向room发，无论room里有多少人，都只产生一条kafka msg.

### What if client conn broken?

send时，就已经把路由信息写到kafka消息了。
job push时，采用的是kafka里的路由信息，但这时候可能已经发生变化，造成丢消息

