---
title: cache invalidation
date: 2017-05-22 09:52:37
tags: 一致性
---

```
// read data
val = cache.get(key)
if val == nil {
    val = db.get(key)
    cache.put(key, val)
}
return val

// write data
db.put(key, val)
cache.put(key, val)
```



这会造成[dual write conflict](http://funkygao.github.io/2017/05/12/dual-write-conflict/)

如果需要的只是eventaul consistency，那么通过[dbus](https://github.com/funkygao/dbus)来进行cache invalidation是最有效的

https://martinfowler.com/bliki/TwoHardThings.html
