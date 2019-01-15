---
layout: post
title:  Oracle 101
date:   2018-12-21 19:30:00 +0800
categories: tools
tags: oracle
---

## 基本概念

* 数据库名(db_name)  
	数据库的唯一标识，在安装数据库、创建新的数据库、创建数据库控制文件、修改数据结构、备份与恢复数据库时都需要使用到的  
	可以使用`select name from v$database;`或`show parameter db;`查看
* 实例名(instance_name/sid)
	数据库的系统标识  
	可以使用`select instance_name from v$instance;`或`show parameter instance;`查看
* 数据库域名(db_domain)
	多节点时使用，可以为空  
	可以使用`select value from v$parameter where name = 'db_domain';`或`show parameter domain;`查看
* 全局数据库名(db_unique_name)
	db_name + db_domain
* 服务名(service_name)
	db_name + db_domain

* SGA(系统全局区)
	SGA = 数据缓冲区+ 重做日志缓冲区+ 共享池+ 大池+ Java 池+ 流池  
	`show sga;`或`select * from v$sga;`  
	```
	SQL> select * from v$sga; 
	NAME                      VALUE
	-------------------- ----------
	Fixed Size              1337548
	Variable Size        1073743668
	Database Buffers      956301312
	Redo Buffers           10858496
	```
* PGA(用户全局区)
	为每个用户进程连接ORACLE数据库保留的内存

## 基本命令

### 监听

运行`lsnrctl`命令的用户需要拥有`dba`权限

```shell
# 查看监听状态
lsnrctl status
# 启动监听
lsnrctl start
# 关闭监听
lsnrctl stop
```

### ping

```
tnsping
```
