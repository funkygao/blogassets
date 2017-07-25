---
title: Google container evolution
date: 2017-07-19 09:45:30
tags: cloud
---

Babbysitter & GlobalWorkQueue -> cgroup -> Borg -> Omega -> Kubernetes

## Container

容器提供的隔离还不完善，OS无法管理的，容器无法隔离(例如CPU Cache，内存带宽)，为了防止(cloud)恶意用户，需要在容器外加一层VM保护

container = isolation + image

数据中心：机器为中心 -> 应用为中心
在Application与OS之间增加一层抽象：container
container的依赖大部分都存放在image了，除了OS的系统调用
实际上，应用仍然会受OS影响，像/proc, ioctl

### 好处

- 更高的资源利用率
- 监控的是应用，而不是混在一起的机器
- 开发者、管理员不需关心OS
- load balancer不再balance traffic across machines: across application instances
- logs are keyed by application, not machine
- 可以更容易识别是应用的失败还是机器的失败

## 教训

- 不要给container端口号
  Borg做法是物理机上的所有容器共用主机的ip，每个容器分单独的端口号
  Kubernetes为每个pod分配不同ip，network身份=application身份
- 不要给container编号
  用label
- 不用暴露raw state
  减少client端的复杂度
  K8s里，都通过API Service访问状态信息

## 还没有很好解决的问题

- Configuration
- Dependency management
  手工配，最终一定不一致
  自动，无法获取足够的语义信息
