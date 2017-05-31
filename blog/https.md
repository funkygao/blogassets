---
title: https
date: 2017-05-19 08:21:27
tags: ops
---

curl https://baidu.com
How the 270ms passed
```
1 1  0.0721 (0.0721)  C>S  Handshake
      ClientHello
        Version 3.1
        cipher suites
        TLS_EMPTY_RENEGOTIATION_INFO_SCSV
        TLS_DHE_RSA_WITH_AES_256_CBC_SHA
        TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
        TLS_DHE_DSS_WITH_AES_256_CBC_SHA
        TLS_RSA_WITH_AES_256_CBC_SHA
        TLS_RSA_WITH_AES_256_CBC_SHA256
        TLS_DHE_RSA_WITH_AES_128_CBC_SHA
        TLS_DHE_RSA_WITH_AES_128_CBC_SHA256
        TLS_DHE_DSS_WITH_AES_128_CBC_SHA
        TLS_RSA_WITH_RC4_128_SHA
        TLS_RSA_WITH_RC4_128_MD5
        TLS_RSA_WITH_AES_128_CBC_SHA
        TLS_RSA_WITH_AES_128_CBC_SHA256
        TLS_DHE_RSA_WITH_3DES_EDE_CBC_SHA
        TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA
        TLS_RSA_WITH_3DES_EDE_CBC_SHA
        compression methods
                  NULL
1 2  0.1202 (0.0480)  S>C  Handshake
      ServerHello
        Version 3.1
        session_id[32]=
          b3 ea 99 ee 5a 4c 03 e8 e0 74 95 09 f1 11 09 2a
          9d f5 8f 2a 26 7a d3 7f 71 ff dc 39 62 66 b0 f9
        cipherSuite         TLS_RSA_WITH_AES_128_CBC_SHA
        compressionMethod                   NULL
1 3  0.1205 (0.0002)  S>C  Handshake
      Certificate
1 4  0.1205 (0.0000)  S>C  Handshake
      ServerHelloDone
1 5  0.1244 (0.0039)  C>S  Handshake
      ClientKeyExchange
1 6  0.1244 (0.0000)  C>S  ChangeCipherSpec
1 7  0.1244 (0.0000)  C>S  Handshake
1 8  0.1737 (0.0492)  S>C  ChangeCipherSpec
1 9  0.1737 (0.0000)  S>C  Handshake
1 10 0.1738 (0.0001)  C>S  application_data
1 11 0.2232 (0.0493)  S>C  application_data
1 12 0.2233 (0.0001)  C>S  Alert
1    0.2234 (0.0000)  C>S  TCP FIN
1    0.2709 (0.0475)  S>C  TCP FIN
```
