---
title: batch insert(mysql)
date: 2017-05-18 08:46:17
tags: database
---

```
// case1
INSERT INTO T(v) VALUES(1), (2), (3), (4), (5)

// case2
for i=1; i<=5; i++ {
    INSERT INTO T(v) VALUES(i);
}
```

case1和2有什么影响？假设auto_commit

## 好处

- 减少与mysql server的交互
- 减少SQL解析(如果statement则没区别)
- query cache打开时，只会invalidate cache一次，提高cache hit

## 坏处

- 可能变成一个大事务
  batch insert的时候，batch不能太大
