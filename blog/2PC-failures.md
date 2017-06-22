---
title: 2 phase commit failures
date: 2017-06-14 16:34:59
tags: 一致性
---

## Node Failure Models

- fail-stop
  crash and never recover
- fail-recover
  crash and later recover
- byzantine failure

## Cases

2 phase commit，n个节点，那么需要3n个消息交换

- coordinator发送proposal后crash
  - 有的node收到，有的没收到
  - 收到Proposal的node被block forever，它可能已经vote commit了
    不能简单地timeout/abort，因为coordinator可能随时recover并启动phase2 commit
    这个txn就只能blocked by coordinator，cannot make any progress
  - 解决办法
    引入coordinator的watchdog机制，它发现coordinator crash后，接管
    Phase1. 先询问每个participants，已经vote commit还是vote abort还是没有vote
    Phase2. 通知每个participant Commit/Abort
    但仍有局限，如果有个participant crash了，那么Phase1无法确认

- worse case
  coordinator本身也是participant

