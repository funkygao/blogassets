---
title: kafka redesign
date: 2017-05-24 10:21:06
tags: PubSub
---

## Goals

- support many topics
  - needle in haystack
- IO optimization
  - R/W isolation
  - index file leads to random sync write
