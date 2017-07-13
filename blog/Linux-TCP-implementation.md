---
title: Linux 2.6.19 TCP implementation
date: 2017-06-30 07:52:33
tags: misc
---

## read

![read](https://github.com/funkygao/blogassets/blob/master/img/netread.png?raw=true)

- /proc/sys/net/ipv4/tcp_rmem
  TCP recv Buffer
- net.core.netdev_max_backlog
  maximum number of packets queued at a device, which are waiting to be processed by the TCP receiving process
  Socket Backlog, dropped if full

## write

![write](https://github.com/funkygao/blogassets/blob/master/img/netwrite.png?raw=true)

- txqueuelen
  ifconfig eth0 txqueuelen
  
