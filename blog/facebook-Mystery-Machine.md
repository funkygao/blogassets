---
title: facebook Mystery Machine
date: 2017-06-05 09:49:00
tags: cloud
---

## Data Flow

consistent sampling

```
local log --> Scribe --> Hive --> UberTrace --> UI
```

## Log Schema

```
request id
host id
host-local timestamp
unique event label(event name, task name)
```

## Timestamp Normalize

- 不考虑local clock skew
- 假设client/server间的RTT是对称的

```
Client        Server
 1|------------>|
  |             |--+1.1
  |             |  | logic
  |             |<-+1.2
 2|<------------|

1.2 - 1.1 = 0.1
2 - 1 = 1.0
RTT = (1.0 - 0.1)/2 = 0.45

clock(1.1) = 1 + 0.45 = 1.45
clock(1.2) = 1.45 + 0.1 = 1.55

RTT是个经验值，根据大量的trace后稳定下来: 使用最小值
```

![global clock](https://github.com/funkygao/blogassets/blob/master/img/the-mystery-machine.png?raw=true)
