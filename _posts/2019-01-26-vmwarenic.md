---
layout: post
title:  VMware Network Adapter
date:   2019-01-26 19:30:00 +0800
categories: sundry
tags: vmware network
---

`VMware`虚拟机有三种网络模式: 桥接(Bridged), 网络地址转换(NAT), 主机(Host-only)。  
`VMware workstation`安装好之后会多出两个网络适配器，分别是`VMware Network Adapter VMnet1`和`VMware Network Adapter VMnet8`，其中VMnet1用于Host-only模式，Vmnet8用于NAT模式。VMnet8和VMnet1提供DHCP服务。

## Birdged

桥接模式下的虚拟机相当于与宿主机处于同一局域网中的一台独立机，需要为其手动配置IP与子网掩码。  
局域网中的其它电脑可以直接与虚拟机互相通信。如果你建立的虚拟机需要为局域网中其它用户提供服务，
则应该选择桥接模式。

从技术来说，相当于在宿主机前端加设了一个虚拟交换机，宿主与虚拟机共享此虚拟机

## NAT(Network Address Translation)

NAT模式下，虚拟机通过宿主机来访问外部网络。外部网络只能看到宿主机，无法看到虚拟机。  
由于NAT有`DHCP`服务，所以使用此模式不用进行任何配置即可访问外部网络。

`NAT`模式正常运行需要宿主机的`VMware DHCP Service`和`VMware NAT Service`两个服务处于运行当中

## Host-only

Host-only模式下，虚拟机与宿主机处于一个与外部网络完全独立的虚拟局域网中，虚拟机无法访问外部网络，外部网络也无法看到虚拟机

## Summary

从网络可见性上来说：  
* Bridged模式下虚拟机与外部网络相互可见
* NAT模式下虚拟机可以看到外部网络但外部网络看不见虚拟机
* Host-only模式下虚拟机与外部网络相互隔离不可见

## 配置

我的一台电脑上用`VMware`创建虚拟机后选择`NAT`模式发现并不能访问外部网络，
检查发现是`VMware DHCP Service`与`VMware NAT Service`两个服务没有开启，但手动启动也无法启动。  
进一步发现电脑上连`VMnet1`与`VMnet8`这两个适配器都没有。重装也无济于事。

其实这是需要使用`VMware`的`Virtual Network Editor`(虚拟网络编辑器)手动添加`VMnet1`与`VMnet8`就可以了

![Virtual_Network_Editor](https://i.imgur.com/rLqAtTL.png)
