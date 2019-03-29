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

## 运行

```shell
systemctl start nginx
```

