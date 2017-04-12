---
layout: post
title:  Chapter 3 Of Names and Address families
date:   2017-04-11 21:20:00 +0800
categories:
  - Reading
  - TCP IP sockets in C by Michael J Donahoo
---

`Socket API`只接受`IP`地址作为目标地址，但是人类并不善于记忆数值型变量。
但我们可以使用服务器名代替，然后转换成`IP`地址再使用  

再运行工程中动态选择`IPv4`或`IPv6`

## Mapping Names to Numbers

使用`name`而不使用数值型`address`不仅是因为`name`容易被人类使用，
而且当服务的`address`变更时，使用它服务的程序无需做任何改动。  
要做到这点，则需要`name`到`address`的一个关系映射。  

有两种方式实现映射：`Domain Name System(DNS)`和`local configuration databases`

### Accessing the Name Service

可以使用如下函数来获取`name service`

```c
int getaddrinfo (const char* hostStr, const char* serviceStr,
                 const struct addrinfo* hints, struct addrinfo** results);
```

前两个参数分别表示`host name / address`和`service name or port number`。  
第三个参数表示返回值的类型  
第四个参数是返回结果，它一个链表。每个节点都包含一个可以用于`socket`连接的完成信息  
返回值：成功返回`0`，否则为一个`非0`值  

```c
struct addrinfo {
  int ai_flags;             // Flags to control info resolution
  int ai_family;            // Family:  AF_INET, AF_INET6, AF_UNSPEC
  int ai_socktype;          // Socket type:  SOCK_STREAM, SOCK_DGRAM
  int ai_protocol;          // Protocol: 0 (default) or IPPROTO_XXX
  socklen_t ai_addrlen;     // Length of socket address ai_addr
  struct sockaddr* ai_addr; // Socket address for socket
  char* ai_canonname;       // Canonical name
  struct addrinfo* ai_next; // Next addrinfo in linked list
};
```

使用`getaddrinfo()`后需要使用另外两个函数：

```c
void freeaddrinfo(struct addrinfo* addrList);
const char *gai_strerror(int errorCode);
```

`getaddrinfo()`的返回结果是一个动态分配的链表，
所以在使用完之后必须调用`freeaddrinfo()`函数进行销毁，否则将造成内存泄漏。  
如果`getaddrinfo()`返回一个非0值，则可将改值传递给`gai_strerror()`从而获取一个关于错误的字符串描述


通过`getaddrinfo()`获取的是一个`socket address`链表。
之所以是一个链表是因为一个`host`可能有多个`IP`，`host`上的一个服务可能支持多种通信协议。
而`getaddrinfo()`可以将他们的不同组合全部获取出来。  
而`getaddrinfo()`的第三个参数则可以限定返回的`socket`类型，从而便于使用，而不必在结果中搜索想要使用的特定类型

`GetAddrInfo.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netdb.h>
#include "Practical.h"

int main(int argc, char* argv[]) {

  if (argc != 3) // Test for correct number of arguments
    DieWithUserMessage("Parameter(s)", "<Address/Name> <Port/Service>");

  char* addrString = argv[1];   // Server address/name
  char* portString = argv[2];   // Server port/service

  // Tell the system what kind(s) of address info we want
  struct addrinfo addrCriteria;                   // Criteria for address match
  memset(&addrCriteria, 0, sizeof(addrCriteria)); // Zero out structure
  addrCriteria.ai_family = AF_UNSPEC;             // Any address family
  addrCriteria.ai_socktype = SOCK_STREAM;         // Only stream sockets
  addrCriteria.ai_protocol = IPPROTO_TCP;         // Only TCP protocol

  // Get address(es) associated with the specified name/service
  struct addrinfo* addrList; // Holder for list of addresses returned
  // Modify servAddr contents to reference linked list of addresses
  int rtnVal = getaddrinfo(addrString, portString, &addrCriteria, &addrList);
  if (rtnVal != 0)
    DieWithUserMessage("getaddrinfo() failed", gai_strerror(rtnVal));

  // Display returned addresses
  for (struct addrinfo* addr = addrList; addr != NULL; addr = addr->ai_next) {
    PrintSocketAddress(addr->ai_addr, stdout);
    fputc('\n', stdout);
  }

  freeaddrinfo(addrList); // Free addrinfo allocated in getaddrinfo()

  exit(0);
}
```

### Detail

`addrinfo`中的`ai_flags`可以使用的值，这些值可以通过`|`进行组合，放入第三个参数中作为筛选条件

 * `AI_PASSIVE` If hostStr is NULL when this flag is set, any returned addrinfos will have their addresses set to the appropriate “any” address constant—inaddr_any (IPv4) or in6addr_any_init (IPv6).
* `AI_CANONNAME` Just as one name can resolve to many numeric addresses, multiple names can resolve to the same IP address. However, one name is usually defined to be the official (“canonical”) name. By setting this flag in ai_flags, we instruct getaddrinfo() to return a pointer to the canonical name (if it exists) in the ai_canonname field of the first struct addrinfo of the linked list.
* `AI_NUMERICHOST` This flag causes an error to be returned if hostStr does not point to a string in valid numeric address format. Without this flag, if the hostStr parameter points to something that is not a valid string representation of a numeric address, an attempt will be made to resolve it via the name system; this can waste time and bandwidth on useless queries to the name service. If this flag is set, a given valid address string is simply converted and returned, a la inet_pton().
* `AI_ADDRCONFIG` If set, getaddrinfo() returns addresses of a particular family only if the system has an interface configured for that family. So an IPv4 address would be returned only if the system has an interface with an IPv4 address, and similarly for IPv6.
AI_V4MAPPED If the ai_family field contains af_inet6, and no matching IPv6 addresses are found, then getaddrinfo() returns IPv4-mapped IPv6 addresses. This technique can be used to provide limited interoperation between IPv4-only and IPv6 hosts.

## Write Address Generic Code

`PrintSocketAddress()`

```c
void PrintSocketAddress(const struct sockaddr* address, FILE* stream) {
  // Test for address and stream
  if (address == NULL || stream == NULL)
    return;

  void* numericAddress; // Pointer to binary address
  // Buffer to contain result (IPv6 sufficient to hold IPv4)
  char addrBuffer[INET6_ADDRSTRLEN];
  in_port_t port; // Port to print
  // Set pointer to address based on address family
  switch (address->sa_family) {
  case AF_INET:
    numericAddress = &((struct sockaddr_in*) address)->sin_addr;
    port = ntohs(((struct sockaddr_in*) address)->sin_port);
    break;
  case AF_INET6:
    numericAddress = &((struct sockaddr_in6*) address)->sin6_addr;
    port = ntohs(((struct sockaddr_in6*) address)->sin6_port);
    break;
  default:
    fputs("[unknown type]", stream);    // Unhandled type
    return;
  }
  // Convert binary to printable address
  if (inet_ntop(address->sa_family, numericAddress, addrBuffer,
      sizeof(addrBuffer)) == NULL)
    fputs("[invalid address]", stream); // Unable to convert
  else {
    fprintf(stream, "%s", addrBuffer);
    if (port != 0)                // Zero not valid in any socket addr
      fprintf(stream, "-%u", port);
  }
}
```

### Generic TCP Client

`TCPClientUtility.c`

```c
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include "Practical.h"

int SetupTCPClientSocket(const char* host, const char* service) {
  // Tell the system what kind(s) of address info we want
  struct addrinfo addrCriteria;                   // Criteria for address match
  memset(&addrCriteria, 0, sizeof(addrCriteria)); // Zero out structure
  addrCriteria.ai_family = AF_UNSPEC;             // v4 or v6 is OK
  addrCriteria.ai_socktype = SOCK_STREAM;         // Only streaming sockets
  addrCriteria.ai_protocol = IPPROTO_TCP;         // Only TCP protocol

  // Get address(es)
  struct addrinfo* servAddr; // Holder for returned list of server addrs
  int rtnVal = getaddrinfo(host, service, &addrCriteria, &servAddr);
  if (rtnVal != 0)
    DieWithUserMessage("getaddrinfo() failed", gai_strerror(rtnVal));

  int sock = -1;
  for (struct addrinfo* addr = servAddr; addr != NULL; addr = addr->ai_next) {
    // Create a reliable, stream socket using TCP
    sock = socket(addr->ai_family, addr->ai_socktype, addr->ai_protocol);
    if (sock < 0)
      continue;  // Socket creation failed; try next address

    // Establish the connection to the echo server
    if (connect(sock, addr->ai_addr, addr->ai_addrlen) == 0)
      break;     // Socket connection succeeded; break and return socket

    close(sock); // Socket connection failed; try next address
    sock = -1;
  }

  freeaddrinfo(servAddr); // Free addrinfo allocated in getaddrinfo()
  return sock;
}
```

`TCPEchoClient.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include "Practical.h"

int main(int argc, char* argv[]) {

  if (argc < 3 || argc > 4) // Test for correct number of arguments
    DieWithUserMessage("Parameter(s)",
        "<Server Address/Name> <Echo Word> [<Server Port/Service>]");

  char* server = argv[1];     // First arg: server address/name
  char* echoString = argv[2]; // Second arg: string to echo
  // Third arg (optional): server port/service
  char* service = (argc == 4) ? argv[3] : "echo";

  // Create a connected TCP socket
  int sock = SetupTCPClientSocket(server, service);
  if (sock < 0)
    DieWithUserMessage("SetupTCPClientSocket() failed", "unable to connect");

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
    // Receive up to the buffer size (minus 1 to leave space for
    // a null terminator) bytes from the sender
    numBytes = recv(sock, buffer, BUFSIZE - 1, 0);
    if (numBytes < 0)
      DieWithSystemMessage("recv() failed");
    else if (numBytes == 0)
      DieWithUserMessage("recv()", "connection closed prematurely");
    totalBytesRcvd += numBytes; // Keep tally of total bytes
    buffer[numBytes] = '\0';    // Terminate the string!
    fputs(buffer, stdout);      // Print the buffer
  }

  fputc('\n', stdout); // Print a final linefeed

  close(sock);
  exit(0);
}
```

### Generic TCP Server

`SetupTCPServerSocket()`

```c
int SetupTCPServerSocket(const char* service) {
  // Construct the server address structure
  struct addrinfo addrCriteria;                   // Criteria for address match
  memset(&addrCriteria, 0, sizeof(addrCriteria)); // Zero out structure
  addrCriteria.ai_family = AF_UNSPEC;             // Any address family
  addrCriteria.ai_flags = AI_PASSIVE;             // Accept on any address/port
  addrCriteria.ai_socktype = SOCK_STREAM;         // Only stream sockets
  addrCriteria.ai_protocol = IPPROTO_TCP;         // Only TCP protocol

  struct addrinfo* servAddr; // List of server addresses
  int rtnVal = getaddrinfo(NULL, service, &addrCriteria, &servAddr);
  if (rtnVal != 0)
    DieWithUserMessage("getaddrinfo() failed", gai_strerror(rtnVal));

  int servSock = -1;
  for (struct addrinfo* addr = servAddr; addr != NULL; addr = addr->ai_next) {
    // Create a TCP socket
    servSock = socket(addr->ai_family, addr->ai_socktype,
        addr->ai_protocol);
    if (servSock < 0)
      continue;       // Socket creation failed; try next address

    // Bind to the local address and set socket to listen
    if ((bind(servSock, addr->ai_addr, addr->ai_addrlen) == 0) &&
        (listen(servSock, MAXPENDING) == 0)) {
      // Print local address of socket
      struct sockaddr_storage localAddr;
      socklen_t addrSize = sizeof(localAddr);
      if (getsockname(servSock, (struct sockaddr*) &localAddr, &addrSize) < 0)
        DieWithSystemMessage("getsockname() failed");
      fputs("Binding to ", stdout);
      PrintSocketAddress((struct sockaddr*) &localAddr, stdout);
      fputc('\n', stdout);
      break;       // Bind and listen successful
    }

    close(servSock);  // Close and try again
    servSock = -1;
  }

  // Free address list allocated by getaddrinfo()
  freeaddrinfo(servAddr);

  return servSock;
}
```

`AcceptTCPConnection()`

```c
int AcceptTCPConnection(int servSock) {
  struct sockaddr_storage clntAddr; // Client address
  // Set length of client address structure (in-out parameter)
  socklen_t clntAddrLen = sizeof(clntAddr);

  // Wait for a client to connect
  int clntSock = accept(servSock, (struct sockaddr*) &clntAddr, &clntAddrLen);
  if (clntSock < 0)
    DieWithSystemMessage("accept() failed");

  // clntSock is connected to a client!

  fputs("Handling client ", stdout);
  PrintSocketAddress((struct sockaddr*) &clntAddr, stdout);
  fputc('\n', stdout);

  return clntSock;
}
```

`TCPEchoServer.c`

```c
#include <stdio.h>
#include "Practical.h"
#include <unistd.h>

int main(int argc, char* argv[]) {

  if (argc != 2) // Test for correct number of arguments
    DieWithUserMessage("Parameter(s)", "<Server Port/Service>");

  char* service = argv[1]; // First arg:  local port

  // Create socket for incoming connections
  int servSock = SetupTCPServerSocket(service);
  if (servSock < 0)
    DieWithUserMessage("SetupTCPServerSocket() failed", service);

  for (;;) { // Run forever
    // New connection creates a connected client socket
    int clntSock = AcceptTCPConnection(servSock);

    HandleTCPClient(clntSock); // Process client
    close(clntSock);
    break;
  }
  // NOT REACHED
  close(servSock);
}
```

## Get Names form Numbers

```c
int getnameinfo (const struct sockaddr*address, socklen_t addressLength,
                char* node, socklen_t nodeLength, char * service,
                socklen_t serviceLength, int flags);
```

```c
int gethostname(char* nameBuffer, size_t bufferLength);
```
