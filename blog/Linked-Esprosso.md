---
title: Linkedin Esprosso
date: 2017-05-22 08:07:48
tags: database
---

## What

Distributed Document Store

- RESTful API
- MySQL作为存储
- Helix负责集群
- Databus异步replicate不同数据中心commit log
- Schema存放在zookeeper，通过Avro的兼容性实现schema evolution

## References

https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store

https://nonprofit.linkedin.com/content/dam/static-sites/thirdPartyJS/github-gists?e_origin=https://engineering.linkedin.com&e_channel=resource-iframe-embed-4
