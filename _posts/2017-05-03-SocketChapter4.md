---
layout: post
title:  Chapter 4 Using UDP Sockets
date:   2017-05-03 08:14:00 +0800
categories:
  - Reading
  - TCP IP sockets in C by Michael J Donahoo
---

## Sending and Receiving with UDP Sockets

```c
ssize_t sendto(int socket, const void* msg, size_t msgLength, int flags,
               const struct sockaddr* destAddr, socklen_t addrLen);
ssize_t recvfrom(int socket, void* msg, size_t msgLength, int flags,
                 struct sockaddr* srcAddr, socklen_t* addrLen);
```

## Connecting a UDP Socket

也可是线使用`connect()`函数连接`UDP Socket`然后使用`send()`与`recv()`通信
