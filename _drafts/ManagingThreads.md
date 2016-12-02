---
layout: post
title:  "C++ Concurrency 101"
date:   2016-12-03 06:33:00 +0800
categories: Concurrency C++
---

* 启动线程
* 等线程结束or让线程在后台自己运行
* 识别特定线程

## 线程的基本操作

每个C++程序都至少有一个线程，即主线程，入口函数`main()`  
所以自己创建线程时首先要定义一个入口函数，你的线程就从这个函数开始执行  
我们已经知道，当程序接收到`main()`的返回值时，程序就结束了(主线程结束)  
同样，当你定义的线程入口函数返回值时，你的线程就结束了

### 启动一个线程

```c++
void do_some_work();
std::thread my_thread(do_some_work);
```

```c++
class background_task
{
  public:
  void operator()() const
  {
    do_something();
    do_something_else();
  }
}

background_task f;
std::thread my_thread(f);
```
