---
title: Wake on LAN
date: 2017-06-09 08:11:10
tags: cloud
---

WOL是个链路层协议，通过发送UDP广播 magic packet远程power up
BIOS可以打开/关闭该功能

```
type MagicPacket struct {
    header  [6]byte // 0xFF
    payload [16]MACAddress
}
```
