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

### 启动实例

```shell
dbstart
```

or 登录`sqlplus`后执行`startup`

### ping

```
tnsping
```

### 查看字符编码

```sql
select * from nls_database_parameters where parameter ='NLS_CHARACTERSET';
```

[参考](https://blog.csdn.net/angus_17/article/details/7762472)

### spool

将`select`查询结果，格式化写入文件

## Chores

### 密码期限

1. 查看用户用的profile策略，一般是default：  

```sql
select username, profile from dba_users;
```

2. 查看指定概要文件（如default）的密码有效期设置：  

```sql
Select * FROM dba_profiles s Where s.profile='DEFAULT' AND s.resource_name='PASSWORD_LIFE_TIME';
```

3. 将密码有效期由默认的180天修改成“UNLIMITED”：  

```sql
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

4. 已经被报告了密码快要过期的账户必须再改一次密码（需要DBA权限

```sql
alter user username identified by passwd;
```

### ORA-27101: shared memory realm does not exist

ORA-01034 and ORA-27101 normally indicate that the database instance you're attempting to connect to is shut down and that you're not connected as a user who has permission to start it up  

启动或重启oracle实例

```shell
$ sqlplus /nolog
>> conn sys/sys as sysdba
>> shutdown abort
>> startup
```

[reference](https://stackoverflow.com/questions/6555827/ora-01034-oracle-not-available-ora-27101-shared-memory-realm-does-not-exist)

### ORA-00837: Specified value of MEMORY_TARGET greater than MEMORY_MAX_TARGET

错误修改`MEMORY_MAX_TARGET`大小，导致`MEMORY_TARGET < MEMORY_MAX_TARGET`, 致使数据库启动失败。

使用修改前的`pfile`文件启动数据库, 生成新的`spfile`再次启动

```sql
startup pfile='/home/oracle/ora11/admin/orcl11g/pfile/init.ora.5242019144032';
create spfile from memory;
shutdown immediate
startup
```

如果找不到`init.ora`文件，可疑自己生成一个`create pfile='/tmp/init.ora' from spfile;`

### ORA-00845: MEMORY_TARGET not supported on this system

启动实例时报错。表明可用的`/dev/shm`空间小于oracle的`memory_target`

使用`df -h`查看`/dev/shm`空间，登录`oracle`后使用`show parameter mem`(实例启动后才有效)查看`memory_target`大小

如果`/dev/shm`空间太小，则可以修改`/etc/fstab`文件增加其大小，重启电脑后生效

```/etc/fstab
tmpfs	/dev/shm	tmpfs	rw,size=4000m	0	0
```

### 日志

```shell
# 关闭监听日志
lsnrctl set log_status off
```

[refrence](https://www.askmaclean.com/archives/%E8%A7%A3%E5%86%B3linux%E4%B8%8A11g%E7%9A%84ora-00845%E9%94%99%E8%AF%AF.html)

## benchmark wich swingbench


