---
title: TLS Session Resumption
date: 2017-06-27 10:00:17
tags: protocol
---

![TLS](https://github.com/funkygao/blogassets/blob/master/img/round-trip-elimination-tls.png?raw=true)

There are two mechanisms that can be used to eliminate a round trip for subsequent TLS connections (discussed below):

- TLS session IDs
  - ServerHello时，server生成一个32字节的session ID给client，后面的TLS握手client可以在它的ClientHello里发送这个id，server就会restore the cached TLS context and avoid the 2nd round trip of TLS handshake
  - nginx支持该方式
    - ssl_session_cache
    - ssl_session_timeout
- TLS session tickets
  与session IDs类似，只是session信息保存在client

[https dialog](/2017/05/19/https/)
