---
title: nagles and delayed ack
date: 2017-06-23 15:23:47
tags: algorithm
---

## Case

同时开启情况下
```
client.send(1600B) // 1600>1460，defragment into Packet(1460)+Packet(140)
client.sendPacket(1460)
server.recv(1460) // no push, server awaiting the next 140
// delayed ack works, so no ack sent s->c
client.sendPacket(140) // because of nagles and has unacked data, wait till 1) data>=1460 or 2) get ack
                       // i,e. will not send packet(140)
... // server ack delay timeout
server.ack(1460)
client.recv(ack)
client.sendPacket(140)
```

## delayed ack

Linux最小值20ms，它是根据RTO、RTT动态计算出来的

## Nagles

- 第一次发包，无论多大，立即发送
- 只要发出的包都被对端ack了就可以发送了，无需等待
- 如果没有ack，就等buffer里的包凑足MSS一起发，即它只允许1个未ack的包存在于网络，基于字节的“停-等”

```
if there is new data to send
  if the window size >= MSS and available data is >= MSS
        send complete MSS segment now
  else
    if there is unacked data still in the buffer
          enqueue data in the buffer until an ack is received
    else
          send data immediately
    end if
  end if
end if
```

## TCP_CORK vs nagles

cork：塞子

cork是一种加强的nagles算法，但它ignore ack，即使所有ack都已经收到，只要数据包不够大而且时间没到，依然不发送
cork是为了提高网络利用率，nagles是为了避免因为过多小包(payload占header比例过小)引起的网络拥堵
