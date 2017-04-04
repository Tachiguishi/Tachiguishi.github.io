---
layout: post
title:  Chapter 2 Basic TCP Sockets
date:   2017-03-15 21:20:00 +0800
categories:
  - Reading
  - TCP IP sockets in C by Michael J Donahoo
---

## IPv4 TCP Client

典型的TCP客户端包含下面4个操作：  
- 使用`socket()`创建`TCP socket`
- 使用`connect()`建立与服务端的连接
- 使用`send(), recv()`进行通信
- 使用`close()`关闭连接

`TCPEchoClient4.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "Practical.h"

int main(int argc, char * argv[]) {

  if (argc < 3 || argc > 4) // Test for correct number of arguments
    DieWithUserMessage("Parameter(s)",
        "<Server Address> <Echo Word> [<Server Port>]");

  char* servIP = argv[1];     // First arg: server IP address (dotted quad)
  char* echoString = argv[2]; // Second arg: string to echo

  // Third arg (optional): server port (numeric).  7 is well-known echo port
  in_port_t servPort = (argc == 4) ? atoi(argv[3]) : 7;

  // Create a reliable, stream socket using TCP
  int sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
  if (sock < 0)
    DieWithSystemMessage("socket() failed");

  // Construct the server address structure
  struct sockaddr_in servAddr;            // Server address
  memset(&servAddr, 0, sizeof(servAddr)); // Zero out structure
  servAddr.sin_family = AF_INET;          // IPv4 address family
  // Convert address
  int rtnVal = inet_pton(AF_INET, servIP, &servAddr.sin_addr.s_addr);
  if (rtnVal == 0)
    DieWithUserMessage("inet_pton() failed", "invalid address string");
  else if (rtnVal < 0)
    DieWithSystemMessage("inet_pton() failed");
  servAddr.sin_port = htons(servPort);    // Server port

  // Establish the connection to the echo server
  if (connect(sock, (struct sockaddr *) &servAddr, sizeof(servAddr)) < 0)
    DieWithSystemMessage("connect() failed");

  size_t echoStringLen = strlen(echoString); // Determine input length

  // Send the string to the server
  ssize_t numBytes = send(sock, echoString, echoStringLen, 0);
  if (numBytes < 0)
    DieWithSystemMessage("send() failed");
  else if (numBytes != echoStringLen)
    DieWithUserMessage("send()", "sent unexpected number of bytes");

  // Receive the same string back from the server
  unsigned int totalBytesRcvd = 0; // Count of total bytes received
  fputs("Received: ", stdout);     // Setup to print the echoed string
  while (totalBytesRcvd < echoStringLen) {
    char buffer[BUFSIZE]; // I/O buffer
    /* Receive up to the buffer size (minus 1 to leave space for
     a null terminator) bytes from the sender */
    numBytes = recv(sock, buffer, BUFSIZE - 1, 0);
    if (numBytes < 0)
      DieWithSystemMessage("recv() failed");
    else if (numBytes == 0)
      DieWithUserMessage("recv()", "connection closed prematurely");
    totalBytesRcvd += numBytes; // Keep tally of total bytes
    buffer[numBytes] = '\0';    // Terminate the string!
    fputs(buffer, stdout);      // Print the echo buffer
  }

  fputc('\n', stdout); // Print a final linefeed

  close(sock);
  exit(0);
}
```

## IPv4 TCP Server

典型的服务端包含以下4个操作：  
- 使用`socket()`创建`TCP socket`
- 使用`bind()`为`socket`分配端口号
- 使用`listen()`进入监听状态，表示允许制定端口的连接
- 重复如下操作：
  - 调用`accept()`从与客户端的连接中获取一个新的`socket`
  - 使用`send(), recv()`通过接受的`socket`和客户端通信
  - 使用`close()`关闭连接

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "Practical.h"

static const int MAXPENDING = 5; // Maximum outstanding connection requests

int main(int argc, char * argv[]) {

  if (argc != 2) // Test for correct number of arguments
    DieWithUserMessage("Parameter(s)", "<Server Port>");

  in_port_t servPort = atoi(argv[1]); // First arg:  local port

  // Create socket for incoming connections
  int servSock; // Socket descriptor for server
  if ((servSock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0)
    DieWithSystemMessage("socket() failed");

  // Construct local address structure
  struct sockaddr_in servAddr;                  // Local address
  memset(&servAddr, 0, sizeof(servAddr));       // Zero out structure
  servAddr.sin_family = AF_INET;                // IPv4 address family
  servAddr.sin_addr.s_addr = htonl(INADDR_ANY); // Any incoming interface
  servAddr.sin_port = htons(servPort);          // Local port

  // Bind to the local address
  if (bind(servSock, (struct sockaddr*) &servAddr, sizeof(servAddr)) < 0)
    DieWithSystemMessage("bind() failed");

  // Mark the socket so it will listen for incoming connections
  if (listen(servSock, MAXPENDING) < 0)
    DieWithSystemMessage("listen() failed");

  for (;;) { // Run forever
    struct sockaddr_in clntAddr; // Client address
    // Set length of client address structure (in-out parameter)
    socklen_t clntAddrLen = sizeof(clntAddr);

    // Wait for a client to connect
    int clntSock = accept(servSock, (struct sockaddr*) &clntAddr, &clntAddrLen);
    if (clntSock < 0)
      DieWithSystemMessage("accept() failed");

    // clntSock is connected to a client!

    char clntName[INET_ADDRSTRLEN]; // String to contain client address
    if (inet_ntop(AF_INET, &clntAddr.sin_addr.s_addr, clntName,
        sizeof(clntName)) != NULL)
      printf("Handling client %s/%d\n", clntName, ntohs(clntAddr.sin_port));
    else
      puts("Unable to get client address");

    HandleTCPClient(clntSock);
  }
  // NOT REACHED
}
```

```c
void HandleTCPClient(int clntSocket) {
  char buffer[BUFSIZE]; // Buffer for echo string

  // Receive message from client
  ssize_t numBytesRcvd = recv(clntSocket, buffer, BUFSIZE, 0);
  if (numBytesRcvd < 0)
    DieWithSystemMessage("recv() failed");

  // Send received string and receive again until end of stream
  while (numBytesRcvd > 0) { // 0 indicates end of stream
    // Echo message back to client
    ssize_t numBytesSent = send(clntSocket, buffer, numBytesRcvd, 0);
    if (numBytesSent < 0)
      DieWithSystemMessage("send() failed");
    else if (numBytesSent != numBytesRcvd)
      DieWithUserMessage("send()", "sent unexpected number of bytes");

    // See if there is more data to receive
    numBytesRcvd = recv(clntSocket, buffer, BUFSIZE, 0);
    if (numBytesRcvd < 0)
      DieWithSystemMessage("recv() failed");
  }

  close(clntSocket); // Close client socket
}
```

## Creating and Destroying Sockets

要使用`TCP`或`UDP`通信，必须请求操作系统创建一个`socket`实例。通过`socket()`函数进行创建

```c
int socket(int domain, int type, protocol);
```

第一个参数表明`socket`进行通信的`domain`。我们这里只需要使用到`IPv4(AF_INET)`和`IPv6(AF_INET6)`  
第二个参数表明`socket`的`type`。`SOCK_STREAM`表明数据是以可信的`byte-stream`方式传递，
`SOCK_DGRAM`数据是以效率优先(`best-effort`)的方式传递  
第三个参数表明`socket`的端对端`end-to-end`通信协议。`IPPROTO_TCP`或`IPPROTO_UDP`  

`socket()`返回一个句柄用于操作`socket`，其本质就是一个文件句柄。如果返回一个正数则表明创建成功，
`-1`表明创建失败。

```c
int close(int socket);
```

`close()`成功返回`0`，失败返回`-1`。`close()`会关闭通信并回收分配给相应`socket`的资源

## Specifying Addresses

### Generic Addresses

`Socket API`定义个`sockaddr`数据结构用来描述`address`

```c
struct sockaddr{
  sa_family_t sa_family;    // address family (e.g. AF_INET)
  char sa_data[14];         // family-specific address information
};
```

### IPv4 Addresses

```c
struct in_addr{
  uint32_t s_addr;          // internet address (32 bits)
};

struct sockaddr_in{
  sa_family_t sin_family;   // internet portocol (AF_INET)
  in_port_t sin_port;       // address port (16 bits)
  struct in_addr sin_addr;  // IPv4 address (32 bits)
  char sin_zero[8];         // Not used
}
```

### IPv6 Addresses

```c
struct in6_addr{
  uint32_t s_addr[16];      // internet address (128 bits)
};

struct sockaddr_in6{
  sa_family_t sin6_family;  // internet protocol (AF_INET6)
  in_port_t sin6_port;      // address port (16 bits)
  uint32_t sin6_flowinfo;   // flow information
  struct in6_addr sin6_addr;// IPv6 address (128 bits)
  uint32_t sin6_scope_id;   // scope identifier
};
```

### Generic Address Storage

`sockaddr_in6`结构体的长度大于`sockaddr`的长度。
而在使用`address`时均需要将其它特定类型的地址结构转换成`sockaddr`。
为了应对不同地址结构体的大小区别，我们使用`sockaddr_storage`结构体进行存储

```c
struct sockaddr_storage{
  sa_family_t
  ...
  // padding and fields to get correct length and alignment
  ...
}
```

### Binary/String Address Conversion

`socket`函数使用的`address`是数字形式的(二进制形式)，但这不便于人类阅读。人类通常使用字符串形式的`address`。
可是使用`inet_pton()`函数将字符串地址转换成数字形。(`pton = printable to numeric`)

```c
int inet_pton(int addressFamily, const char* src, void* dst);
```

第一个参数指定`address family`，这里我们只需用到`AF_INET`和`AF_INET6`  
第二个参数指定需要进行转化的字符串形`address`  
第三个参数为转化后结果保存位置，其所占用的空间大小必须足够保存结果(at least 4 bytes for IPv4 and 16 bytes for IPv6)  

`inet_pton()`返回`1`则表示转换成功，则可以通过`dst`获取转换结果  
返回`0`则表示`src`参数并不是一个有效的字符串地址  
返回`-1`则表示`address family`未知

```c
const char* inet_ntop(int addressFamily, const void* src, char* dst, socklen_t dstBytes);
```

第三和第四个参数分别表示转换后字符串的首地址和占用空间。
系统中定义有`INET_ADDRSTRLEN`和`INET6_ADDRSTRLEN`分别表示`IPv4`和`IPv6`字符串`address`最大所占用的空间大小  
如果转换成功，则返回转换后字符串的首地址(即第三个参数`dst`的地址)，如果转换失败则返回`NULL`

### Getting a Socket's Associated Addresses

一个`socket`连接同时包含本机和远程`address`。
我们可以分别使用`getsockname()`和`getpeername()`获取本机和远程`address`。获取的结果是个`sockaddr`结构体

```c
int getpeername(int socket, struct sockaddr* remoteAddress, socklen_t* addressLength);
int getsockname(int socket, struct sockaddr* localAddress, socklen_t* addressLength);
```

## Connecting a Socket

客户端与服务端的区别在于：客户端发起连接，服务端被动接受连接请求。

```c
int connect(int socket, const struct sockaddr* foreignAddress, socklen_t addressLength);
```

## Binding to an Address

服务端要通过`bind()`定义服务端用于连接的`address`是多少

```c
int bind(int socket, struct sockaddr* localAddress, socklen_t addressSize);
```

成功返回`0`，失败返回`-1`  
如果想要服务端响应`host`上所有`address`，
可以将参数`localAddress`设为`INADDR_ANY`或`in6addr_any`，即`wildcard address`。  
如果实在初始化时将`in6_addr`设为`wildcard address`可以使用`IN6ADDR_ANY_INIT`。  

注意：`INADDR_ANY`是以`host byte order`定义的，所以在传递给`bind()`作参数前必须转化成`network byte order`。
可以使用`htonl()`函数进行转化。而`in6addr_any`和`IN6ADDR_ANY_INIT`本身就是`network byte order`，不需要转化  

如果将端口号设为`0`传递给`bind()`，则系统会自动选取一个尚未被使用的端口号来进行连接

## Handling Incoming Connections

当通过`bind()`确认服务端的`address`之后，即可让服务端的`socket`进入监听状态，等待客户端连接请求，这需要使用`listen()`

```c
int listen(int socket, int queueLimit);
```

成功返回`0`，失败返回`-1`。

服务端接受和发送数据并不是使用服务端的`socket`，而是使用客户端的`socket`。
所以首先要获取客户端的`socket`，这就需要使用`accept()`函数

```c
int accept(int socket, sockaddr* clientAddress, socklen_t* addressLength);
```

`accept()`从`socket`队列中获取一个通信数据，如果队列为空，则程序会柱塞在这里直到接受到一个通信。
如果获取成功，则`clientAddress`和`addressLength`会被赋予客户端的`address`和长度。
并将客户端的`socket`作为返回值返回  
如果失败则返回`-1`  

第一个参数`socket`时服务器端的`socket`，其一直保持监听状态而没有和客户端连接

```c
struct sockaddr_in clntAddr; // Client address
// Set length of client address structure (in-out parameter)
socklen_t clntAddrLen = sizeof(clntAddr);
// Wait for a client to connect
int clntSock = accept(servSock, (struct sockaddr *) &clntAddr, &clntAddrLen);
```

## Communication

一旦`socket`处于连接状态，则可以开始发送和接受数据。
客户端连接的`socket`时通过调用`connect()`函数建立的，而服务端连接的`socket`是通过`accept()`函数返回值获取的。  
当`socket`连接后，服务端和客户端的区别便不复存在，运用同样的方式使用`send()`和`recv()`进行数据交换

```c
ssize_t send(int socket, const void* msg, size_t msgLength, int flags);
ssize_t recv(int socket, void* rcvBuffer, size_t bufferLength, int flags)
```
