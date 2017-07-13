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

在facebook数据中心，开辟了一块单独的“冷存储”，专门保存不再查看的照片、视频(通常都是10年前的)，由于
meta/data分离，用户想看，会wake这些存储设备
