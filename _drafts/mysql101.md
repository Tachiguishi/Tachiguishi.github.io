---
layout: post
title:  MySQL 101
date:   2018-12-14 19:30:00 +0800
categories: tools
tags: mysql
---

## 安装

[参考](https://www.if-not-true-then-false.com/2010/install-mysql-on-fedora-centos-red-hat-rhel/)

### Fedora28

```shell
sudo dnf install https://dev.mysql.com/get/mysql80-community-release-fc28-1.noarch.rpm
sudo dnf install mysql-community-server
sudo systemctl start mysqld.service
sudo systemctl enable mysqld.service
# Get Your Generated Random root Password
grep 'A temporary password is generated for root@localhost' /var/log/mysqld.log |tail -1
# Start MySQL Secure Installation
/usr/bin/mysql_secure_installation
# new root passwd: fedora@syh0
```

```mysql
CREATE DATABASE webdb;
CREATE USER 'webdb_user' IDENTIFIED BY 'password123';
GRANT ALL ON webdb.* TO 'webdb_user';
FLUSH PRIVILEGES;
```

## 配套工具

### [pt-query-digest](https://www.percona.com/doc/percona-toolkit/LATEST/pt-query-digest.html)

Analyze MySQL queries from logs, processlist, and tcpdump  
用于分析查询日志，定位影响性能的是那些查询语句


## issues

### 监听地址

`mysql`默认监听`127.0.0.1`, 只需要修改其配置文件即可

`/etc/mysql/mysql.conf.d/mysqld.cnf`

```conf
bind-address = 0.0.0.0
```

### 表名区分大小写

由`lower_case_table_names`配置，linux中默认为0

0 库表名以创建语句中指定的字母大小写存储在磁盘上，名称比较区分大小写。
1 库表名以小写形式存储在磁盘上，名称比较不区分大小写。MySQL 在存储和查找时将所有表名转换为小写。此行为也适用于数据库名称和表别名。
2 库表名以创建语句中指定的字母大小写存储在磁盘上，但是 MySQL 在查找时将它们转换为小写。名称比较不区分大小写。
