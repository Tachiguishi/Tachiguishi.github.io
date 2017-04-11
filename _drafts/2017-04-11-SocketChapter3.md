---
layout: post
title:  Chapter 2 Of Names and Address families
date:   2017-03-15 21:20:00 +0800
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
