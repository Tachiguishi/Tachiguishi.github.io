---
layout: post
title:  Ngrok
date:   2019-08-27 19:15:00 +0800
categories: tools
tags: ngrok
---

`ngrok`是一个反向代理程序，用于实现内网穿透。可以捕获并分析`http/https`流量，并支持`tcp`代理  
`1.x`开源，[repo地址](https://github.com/inconshreveable/ngrok)  
`2.x`不开源，[官网](https://ngrok.com/)  
故这里只介绍如何使用`1.x`自建服务

## Install

`ngrok`使用`go`编写，所以需要有`go`环境，且`go`版本需要`>=1.8`

```shell
mkdir ~/go/src/github.com/inconshreveable
cd ~/go/src/github.com/inconshreveable
git clone --depth=1 https://github.com/inconshreveable/ngrok
cd ngrok
make
```

如果`make`顺利编译，则回在`ngrok`目录下生成程序`go-bindata`, `ngrok`, `ngrokd`  
其中`ngrokd`为服务端程序，`ngrok`为客户端程序

## 生成属于自己的证书

```
NGROK_DOMAIN="example.com"
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

cd ngrok
cp server.key assets/server/tls/snakeoil.key
cp server.crt assets/server/tls/snakeoil.crt
cp base.pem assets/client/tls/ngrokroot.crt
```

## 运行服务端

假设你拥有域名`yourdomain.com`,

```shell
ngrokd -domain=yourdomain.com -httpAddr=:2080 -httpsAddr=:2143 -tunnelAddr=:2280
ngrokd -tlsKey=./tls/snakeoil.key -tlsCrt=./tls/snakeoil.crt -tunnelAddr=:2280
```

其中`httpAddr`与`httpsAddr`指定是反向代理`http`与`https`时所使用的端口，如果不指定则默认为80/443  
`tunnelAddr`是服务端监听用于与客户端通信的端口，仅供`ngrok`自己使用

## 运行客户端

### 配置文件

`ngrok.cfg`(配置使用`YAML`语法，缩进需要使用空格，不能使用`tab`)

```yaml
server_addr: "yourdomain.com:2280"
trust_host_root_certs: false
tunnels:
 tcpapp:
  remote_port: 2323
  proto:
   tcp: 2000
 webapp:
  subdomain: "www"
  proto:
   http: 80
```

服务端可以指定证书的路径，但客户端好像不可以

`server_addr`字段配置服务端的地址与端口  
`trust_host_root_certs`字段在使用`https`协议时作用  
`tunnels`配置需要代理的服务，这里有两个`tcpapp`和`webapp`, 两则的名称可以自定义  
`tcpapp`服务使用`tcp`协议，本地监听端口`2000`, `ngrok`服务端监听端口`2323`, 
这样所以发送到`yourdomain.com:2323`的包都会被转发到本地主机的`2000`端口  
`webapp`服务使用`http`协议，本地监听端口`80`, `ngrok`服务端监听端口`2080`, 
服务端监听端口是在`ngrokd`启动时指定的，所以客户端配置中无需指定。 
使用`http`协议必须要配置`subdomain`字段，他会加在你服务端域名前面，
所以服务端会将所有`www.yourdomain.com:2080`的数据转发到本地主机的`80`端口

### 启动

```shell
ngrok -config=ngrok.cfg start tcpapp webapp
```

[reference](https://tsukkomi.org/post/use-ngrok-to-puch-the-nat)
