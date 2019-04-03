---
layout: post
title:  Nginx
date:   2019-03-29 19:15:00 +0800
categories: tools
tags: nginx
---

## 安装

[官网安装教程](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

### CentOS

`/etc/yum.repos.d/nginx.repo`

```conf
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

根据自己的系统的世纪版本修改`$releasever`为`5 (for 5.x) or 6 (for 6.x) or 7`

```shell
yum install nginx
```

修改`nginx`配置文件`/etc/nginx/conf.d/default.conf`

```conf
server_name domain.me;
server_name www.domain.me;
```

## 运行

```shell
systemctl start nginx
```

## TLS证书

### 获取证书

使用`acme.sh`脚本安装

```shell
curl https://get.acme.sh | sh
source ~/.bashrc
acme.sh --issue -d domain.me -d www.domain.me --nginx
# or 制定加密方式
acme.sh --issue -d domain.me -d www.domain.me --nginx -k ec-256
```

如果一切顺利则会产生如下结果

```
Your cert is in  /root/.acme.sh/domain.me/domain.me.cer
Your cert key is in  /root/.acme.sh/domain.me/domain.me.key
The intermediate CA cert is in  /root/.acme.sh/domain.me/ca.cer
And the full chain certs is there:  /root/.acme.sh/domain.me/fullchain.cer
```

### 使用证书

修改`nginx`配置文件`/etc/nginx/conf.d/default.conf`

```conf
listen  443 ssl;
ssl on;
ssl_certificate       /path/fullchain.crt;
ssl_certificate_key   /path/domain.me.key;
ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers           HIGH:!aNULL:!MD5;
```

将`80`端口转向`443`接口

```conf
server {
	listen 80;
	server_name domain.me;
	server_name www.domain.me;
	rewrite ^(.*)$ https://${server_name}$1 permanent;
}
```

重启`nginx`

```shell
systemctl restart nginx
```

### 验证证书

在网站[SSL_Labs](https://www.ssllabs.com/ssltest)中输入你的域名即可检测结果

