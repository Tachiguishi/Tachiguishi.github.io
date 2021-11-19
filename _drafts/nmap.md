---
layout: post
title:  NMAP
date:   2021-10-18 10:44:00 +0800
categories: tools
tags: nmap
---

## PORT SPECIFICATION AND SCAN ORDER

1. `-p <port ranges>`

### Examples

1. nmap -pPort hostname  
检测目标`hostname`的端口`Port`状态: nmap -p22 1.1.1.1

2. nmap -pPort1-Port2 hostname  
检测目标`hostname`的端口区域`Port1-Port2`状态 nmap -p1-1024 1.1.1.1

## HOST DISCOVERY

1. `-sn: Ping Scan - disable port scan`

### Examples

1. nmap -sn 172.21.1.0/24  
通过`ICMP`报文检测局域网`172.21.1.0/24`内哪些IP正在被使用

## SCAN TECHNIQUES

1. `-sS` TCP SYN scan

### Examples

1. nmap -sS 172.21.1.0/24  
通过发送`TCP SYN`检测目标机器的端口状态
