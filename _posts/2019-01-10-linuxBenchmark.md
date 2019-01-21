---
layout: post
title:  Linux Benchmark
date:   2019-01-10 19:30:00 +0800
categories: tools
tags: linux
---

基准测试即通过某一标准流程测试系统性能，常用于测试：  
* 系统是否发挥正常
* 应该用哪个版本的驱动以达到最佳性能
* 系统是否能够胜任某任务

[参考](https://wiki.archlinux.org/index.php/Benchmarking)

## UnixBench

[UnixBench](https://github.com/kdlucas/byte-unixbench)类unix系(Unix，BSD，Linux)统下的性能测试工具，一个开源工具。
测试包含了系统调用、读写、进程、2D、3D、管道、运算、C库等系统基准性能。
它的优点在于提供了对系统性能的一种评价体系，为系统评分，如此方便对系统作对比测试；但在网络性能测试欠缺。

### 编译

为了测试图形需要将`Makefile`中的`GRAPHIC_TESTS = defined`取消注释，并安装`x11`与`openGL`开发库，然后编译`make`。

```shell
## no file <X11/Xlib.h>
dnf install libXext-devel
apt-get install libxext-dev

## no file <GL/gl.h>
dnf install mesa-libGL-devel
apt-get install libgl1-mesa-dev

## undefined reference to 'sincos'
#=> 在`Makefile`的`GL_LIBS`项后添加`-lm`
```

### 运行

```shell
# 执行system测试方法
Run
# 执行graphic测试方法
Run graphics
# 执行system,graphics测试方法
Run gindex
```

> 运行`graphics`测试时需要`root`权限

#### 运行时bug

* `Can't locate Time/HiRes.pm`  
需要安装`perl`

```shell
dnf install perl-devel
```

如果已经安装，则需要单独安装`Time/HiRes`包

```shell
perl -MCPAN -e shell
cpan[2]> install Time::HiRes 
cpan[3]> exit
```

* `can't exec x11perf`  

```shell
yum install xorg-x11-apps
```

### 测试项目

* Dhrystone 2 using register variables  
此项用于测试 string handling，因为没有浮点操作，所以深受软件和硬件设计(hardware and software design)、编译和链接(compiler and linker options)、代码优化(code optimazaton)、对内存的cache(cache memory)、等待状态(wait states)、整数数据类型(integer data types)的影响。

* Double-Precision Whetstone  
这一项测试浮点数操作的速度和效率。这一测试包括几个模块，每个模块都包括一组用于科学计算的操作。覆盖面很广的一系列 c 函数：sin，cos，sqrt，exp，log 被用于整数和浮点数的数学运算、数组访问、条件分支(conditional branch)和程序调用。此测试同时测试了整数和浮点数算术运算。

* Execl Throughput  
此测试考察每秒钟可以执行的 execl 系统调用的次数。 execl 系统调用是 exec 函数族的一员。它和其他一些与之相似的命令一样是 execve() 函数的前端。

* File copy  
测试从一个文件向另外一个文件传输数据的速率。每次测试使用不同大小的缓冲区。这一针对文件 read、write、copy 操作的测试统计规定时间(默认是 10s)内的文件 read、write、copy 操作次数。

* Pipe Throughput  
管道(pipe)是进程间交流的最简单方式，这里的 Pipe throughtput 指的是一秒钟内一个进程可以向一个管道写 512 字节数据然后再读回的次数。需要注意的是，pipe throughtput 在实际编程中没有对应的真实存在。

* Pipe-based Context Switching  
这个测试两个进程(每秒钟)通过一个管道交换一个不断增长的整数的次数。这一点很向现实编程中的一些应用，这个测试程序首先创建一个子进程，再和这个子进程进行双向的管道传输。

* Process Creation  
测试每秒钟一个进程可以创建子进程然后收回子进程的次数(子进程一定立即退出)。process creation 的关注点是新进程进程控制块(process control block)的创建和内存分配，即一针见血地关注内存带宽。一般说来，这个测试被用于对操作系统进程创建这一系统调用的不同实现的比较。

* System Call Overhead  
测试进入和离开操作系统内核的代价，即一次系统调用的代价。它利用一个反复地调用 getpid 函数的小程序达到此目的。

* Shell Scripts  
测试一秒钟内一个进程可以并发地开始一个 shell 脚本的 n 个拷贝的次数，n 一般取值 1，2，4，8。(我在测试时取 1， 8)。这个脚本对一个数据文件进行一系列的变形操作(transformation)。

### 测试结果

在`./results`目录下

## sysbench

### 安装

```shell
# 安装EPEL源
dnf install epel-release
dnf install sysbench
sysbench --version
#=> sysbench 1.0.12
```
