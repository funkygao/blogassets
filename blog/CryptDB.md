---
title: CryptDB
date: 2017-06-26 16:46:21
tags: database
---

一个DB Proxy，对字段名称、记录都加密，Google根据CryptDB的设计开发了Encrypted BigQuery client
仍然存在数据泄露问题

![deployment](http://static.yihaodou.com/up_img/2016/01/14529240805699dcb05b960.jpg)

## Related

- CryptDB enables most DBMS functionality with a performance overhead of under 30%
- Arx is built on top of MongoDB and reports a performance overhead of approximately 10%
- OSPIR-EXT/SisoSPIR support MySQL
- BlindSeer

![db crypt](https://github.com/funkygao/blogassets/blob/master/img/crypdb.jpeg?raw=true)

## References

https://github.com/CryptDB/cryptdb
http://people.csail.mit.edu/nickolai/papers/raluca-cryptdb.pdf
https://people.csail.mit.edu/nickolai/papers/popa-cryptdb-tr.pdf
