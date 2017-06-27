---
title: chain replication
date: 2017-06-27 13:21:43
tags: storage
---

与primary/backup repliation不同，是一种ROWAA(read one, write all available)方法，比quorum有更高的可用性

![chain replication](https://github.com/funkygao/blogassets/blob/master/img/chain.png?raw=true)

## References

https://www.usenix.org/legacy/event/osdi04/tech/full_papers/renesse/renesse.pdf
http://snookles.com/scott/publications/erlang2010-slf.pdf
https://github.com/CorfuDB/CorfuDB/wiki/White-papers
