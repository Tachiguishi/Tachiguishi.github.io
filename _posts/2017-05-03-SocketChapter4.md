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

`sendto()`的前四个参数与`send()`的完全相同，只是在后面添加了目标地址及其长度大小，
因为`UDP`没有`connect()`操作，所以要在发送时指明目标地址  
`recvfrom()`的前四个参数与`recv()`的相同，并在之后添加了两个参数用于接受消息的来源地址。
注意`recvfrom()`中的`addrLen`参数是一个输入／输出参数，在输入时表示`srcAddr`地址缓冲区的大小，
在输出时表示实际复制到地址缓冲区中的地址的大小  

`TCP`与`UDP`的一个区别就是`UDP`会保留边界信息，这就意味着一次`recvfrom()`调用最多只会接受来自
一个`sendto()`发送的数据。而不同次调用的`recvfrom()`接受的数据永远不可能来自同一次`sendto()`发送
(除非使用`MSG_PEEK`标识)。  
而`TCP`中一个`recv()`可能接受的是多个`send()`发送的数据，或多个`recv()`接受来自一个`send()`发送的数据。  

当`send()`返回时，调用者只知道数据被放入的发送缓冲区，数据可能会也可能不会实际传输到目标地址。
由于`UDP`没有错误处理机制，所以不需要缓冲区对发送失败数据进行重传，这就是说当`sendto()`返回时，
数据已经传输或正在传输到目标地址  

在数据被发送到目标服务器但是在`recv()`或`recvfrom()`被调用前，数据被保存在一个`FIFO`的接受缓冲区队列。
所有接受到的数据在这个缓冲区中是一个连续的序列。由于`recvfrom()`要返回数据的源地址，所以`UDP`发送的消息
需要保留边界信息。如果调用`recvfrom()`时使用的接受数据缓冲区大小`n`小于`FIFO`队列中第一数据块的大小，则
`recvfrom()`只会接受到该块数据的前`n`个字节，而剩余的字节则会被自动舍弃且调用着无法感知。  
所以在使用`recvfrom()`时要使用比接受数据大的缓冲区才不会导致数据丢失。
`UDP`中`recvfrom()`可以返回的最大字节数为`65507 bytes`   

如果在`recvfrom()`调用时使用`MSG_PEEK`标识，则在调用后数据必然会保存在`FIFO`接受缓冲区队列中，
这样就可以多次重复获取同一`sendto()`发送的数据。如果在发送数据的头部写入数据块的大小信息，
则可以使用这用方法提前探测数据块的真实大小，然后使用合适的缓冲区大小接受数据，从而保证没有数据丢失

## Connecting a UDP Socket

也可使用`connect()`函数连接`UDP Socket`然后使用`send()`与`recv()`通信
