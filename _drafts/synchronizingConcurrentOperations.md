---
layout: post
title:  Chapter 4 Synchronizing concurrent operations
date:   2017-02-21 22:11:00 +0800
categories:
  - Reading
  - C++ Concurrency in Action by Anthony Williams
---

## waiting for an event or other condition

A线程需要等待B线程执行完某项操作，则可以在共享数据中定义一个标志符，如果B线程完成操作则将其置位。
而A线程则轮询检测该标志符是否被置位

```c++
bool flag;
std::mutex m;

void wait_for_flag()
{
  std::unique_lock<std::mutex> lk(m);
  while(!flag)
  {
    lk.unlock();
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    lk.lock();
  }
}
```

此方法虽然可行，但是轮询周期却很难确定。如果周期太短，则频繁地加锁解锁将影响系统效率。
如果周期太长，则造成A线程不必要的等待时间  
c++标准库提供了相应的解决方案

### waiting for a condition with condition variables
