---
layout: post
title:  "C++ Concurrency 101"
date:   2016-12-03 06:24:00 +0800
categories: Concurrency C++
---

使用 c++11 标准库中的多线程来学习多线程。  

### Hello Word

hello.cpp  
```c++
#include <iostream>
#include <thread>

void hello()
{
    std::cout<<"Hello Concurrent World\n";
}

int main()
{
    std::thread t(hello);
    t.join();
}
```

使用 g++ 编译器编译  

```bash
$ g++ -std=c++11 -lpthread hello.cpp -o Hi
```
