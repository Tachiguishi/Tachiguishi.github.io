---
layout: post
title:  Chapter 5 Send and Receive Data
date:   2017-05-03 12:34:00 +0800
categories:
  - Reading
  - TCP IP sockets in C by Michael J Donahoo
---

## Encoding Integers

由于`socket`通信交换的数据实际为字节序列，所以在解析时需要提前得知各字节代表的是何种类型的数值

### Sizes of Integers

C语言并没有规定`int, short, char, long`等整数类型占用内存的实际字节数，所以在不同平台间通信时很可能会出错。
但是C语言中定义了`int8_t, int16_t, int32_t, int64_t`等类型确定定义整数所占用的字节数

### Byte Ordering

如果一个整数超过了一个字节，则字节的顺序则会有`little-endian`和`big-endian`两种情况。  
网络协议通常使用`big-endian`，所以其又被称为`network byte order`

### Signedness and Sign Extension

### Encoding Integers by Hand

### Wrapping TCP Sockets in Streams

### Structure Overlays: Alignment and Padding

C明确定义了数据结构如何在内存中存储，所以可以在网络间直接传递二进制数据。  
但是由于[C语言字节对其方式](http://www.cnblogs.com/clover-toeic/p/3853132.html)，
编译器可能会在数据结构间添加`padding`，而添加的内容是不确定的。
这很可能引起数据接受错误。  
但是可以通过在数据结构中显示添加`padding`字段来避免编译器来添加`padding`

### Strings and Text

使用字符串作为通行内容非常方便，且很容易在解析后转换成其它类型的数据。  
c中字符串实际为字符数组，对于英文用`char`就已经足够，但是对于某些语言(汉语)，只用`char`远远不够  
`char`为`1 byte, 8 bits`，能显示`128`个字符  
所以为了显示多字节字符c中定义了`wchat_t`类型  
`char`和`wchar_t`之间可以相互转换

```c
#include <stdlib.h>
size_t wcstombs(char* restrict s, const wchar_t* restrict pwcs, size_t n);
size_t mbstowcs(wchar_t* restrict pwcs, const char* restrict s, size_t n);
```

### Bit-Diddling: Encoding Booleans

一个int又8位，可以讲每一位作为`boolean`

## Constructing, Framing, and Parsing Messages
