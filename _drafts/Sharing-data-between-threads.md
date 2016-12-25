---
layout: post
title:  Sharing data between threads
date:   2016-12-13 19:00:00 +0800
categories:
  - Reading
  - C++ Concurrency in Action by Anthony Williams
---

## 线程间数据共享的问题

共享数据的问题都是由于需要修改数据引起的，
如果共享的数据全是只读数据，那么不会有任何问题  
修改数据时很可能会破环数据结构的 invariants(描述数据结构某一属性的变量，如列表所包含的元素个数)

### Race Conditions

在并行程序中是指程序的输出结果依赖于不同进程相关操作的执行顺序，从而导致结果不可控  
(如对一个共享数据，一个线程修改一个线程读取，那么两者的执行顺序不同则结果不同。
而对读取数据的线程来说它无法控制修改线程的执行时机，从而无法控制自己的输出结果)

并不是所有的竞争都会引起错误，只有会破坏 invariants 结构的竞争才会引发问题。  
恶性竞争的发生和时间窗口有很大关系，通常在 debugger 模式和低负荷下很难出现，也很难再现 bug

## 使用互斥量(mutex, mutual exclusion)保护数据

使用 mutex 保护数据是指，一个线程在访问共享数据前先锁住mutex，访问完成后再解锁。
其它线程只有在等改线程解锁后才能去锁mutex然后访问数据。  
mutex也并不是一劳永逸的方法，它并不能解决所有的竞争问题，还有自己引发其它问题

### mutex

```c++
#include <list>
#include <mutex>
#include <algorithm>
#include <iostream>

std::list<int> listTest;
std::mutex listMutex;

void addToList(int value)
{
    std::lock_guard<std::mutex> guard(listMutex);
    listTest.push_back(value);
    return;
}

bool isListContain(int value)
{
    std::lock_guard<std::mutex> guard(listMutex);
    return std::find(listTest.begin(), listTest.end(), value)
            != listTest.end();
}

int main()
{
    addToList(42);
    std::cout<<"contains(1)="<<isListContain(1)
        <<", contains(42)="<<isListContain(42)<<std::endl;
}
```

为了避免手动加锁解锁，可以使用`std::lock_guard`。  
其采用`RAII`结构，在构造函数里调用`lock`，在析构函数里调用`unlock`

### 精心构造代码

仅仅加锁并不能完全保护共享数据，如果外界能够获取共享数据的指针或引用，
则可以无视锁的存在在任何时候访问到数据。  
所以尽量不要使外界可以获取共享数据的指针或引用

```c++
#include <mutex>

class someData
{
private:
    int a;
    str::string b;
public:
    void doSomething(){}
}

class shearData
{
private:
    someData data;
    std::mutex m;
public:
    template<typename Function>
    void process_data(Function func)
    {
        std::lock_guard<std::mutex> g(m);
        func(data);
    }
}

someData* unprotected;

void malicious_function(someData& protected_data)
{
    unprotected = &protected_data;
}

shearData x;

void foo()
{
    x.process_data(malicious_function);
    unprotected->doSomething();
}

int main()
{
    foo();
}
```

### 接口中的竞争条件

## 保护数据的其它方式
