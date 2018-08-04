---
layout:     post
title:      System performance issues with UDP transports
date:       2018-12-7
summary:    New transports such as QUIC are based on UDP sockets. Linux
            ecosystem is more tunned towards TCP performance than for UDP. This blog
            explains the technicalities on why scaling with UDP is a problem using
            userspace sockets.
categories: protocols
---

* REUSE_PORT and issues
* PACKET_RX_RING
* XDP
* ebpf based packet redirection

[Want to comment?](https://github.com/isaacs/github/issues/new?title=UDP-TRANSPORT)
