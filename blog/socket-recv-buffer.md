---
title: socket recv buffer
date: 2017-06-08 11:03:16
tags: algorithm
---

![SO_RCVBUF](https://github.com/funkygao/blogassets/blob/master/img/recvbuf.png?raw=true)

```
BDP * 4 / 3
```

use iperf for testing
```
iperf -s -w 200K   # server
iperf -c localhost # client
```

![iperf](https://github.com/funkygao/blogassets/blob/master/img/iperf.png?raw=true)

```
ethtool -k eth0 # segment offload makes tcpdump see a packet bigger than MTU
net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_congestion_control
net.ipv4.tcp_moderate_rcvbuf

net.ipv4.tcp_slow_start_after_idle
```
