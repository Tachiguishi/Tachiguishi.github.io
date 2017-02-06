---
layout: post
title:  Chapter 3 Sharing data between threads
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
};

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
};

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

仅仅使用mutex并不能完全避免竞争，还需要考虑数据结构和结构。  
考虑有如下数据结构：

```c++
template<typename T, typename Container=std::deque<T>>
class stack
{
public:
  explicit stack(const Container&);
  explicit stack(Container&& = Container());
  template<class Alloc> explicit stack(const Alloc&);
  template<class Alloc> stack(const Container&, const Alloc&);
  template<class Alloc> stack(Container&&, const Alloc&);
  template<class Alloc> stack(stack&&, const Alloc&);

  bool empty() const;
  size_t size() const;
  T& top();
  T const& top() const;
  void push(T const&);
  void push(T&&);
  void pop();
  void swap(stack&&);
};
```

这里的问题在于，如果仅仅在每个借口函数中加锁解锁，则`empty()`和`size()`函数的返回值则不可靠。
因为在函数返回后，其它线程可能有机会改变数据。即如下代码在多线程中则很可能会出错

```c++
stack<int> s;
if(!s.empty())
{
  int const value = s.top();
  s.pop();
  doSomething(value);
}
```

在调用`top()`和`pop()`之间也可能出现竞争。但是如果之间使用`int const value = s.pop()`的方式获取数据也有可能出错。
如果机器内存不足，在调用`pop()`成功后(stack中的数据被删除)，但是为`value`分配空间失败，则会造成数据丢失。

解决：
1. 使用引用参数

```c++
std::vector<int> result;
some_stack.pop(result);
```

1. 避免使用会抛出异常的`copy constructor`或`move constructor`
1. `pop()`返回的是元素的指针
1. 同时使用方法1和2(或3)

多线程安全的`stack`结构实例

```c++
#include <exception>
#include <memory>     // for std::shared_ptr()
#include <mutex>
#include <stack>

struct empty_stack: std::exception
{
  const char* what() const throw();
};

template<typename T>
class threadsafe_stack
{
private:
  std::stack<T> data;
  mutable std::mutex m;

public:
  threadsafe_stack(){}
  threadsafe_stack(const threadsafe_stack& other)
  {
    std::lock_guard<std::mutex> lock(other.m);
    data = other.data;
  }
  threadsafe_stack& operator=(const threadsafe_stack&) = delete;  // 禁止赋值操作

  void push(T new_value)
  {
    std::lock_guard<std::mutex> lock(m);
    data.push(new_value);
  }
  std::shared_ptr<T> pop()
  {
    std::lock_guard<std::mutex> lock(m);
    if(data.empty()) throw empty_stack();
    std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
    data.pop();
    return res;
  }
  void pop(T& value)
  {
    std::lock_guard<std::mutex> lock(m);
    if(data.empty()) throw empty_stack();
    value = data.top();
    data.pop();
  }
  bool empty() const
  {
    std::lock_guard<std::mutex> lock(m);
    return data.empty();
  }
};
```

要合理考虑使用`mutex`的个数，设计锁的粒度(granularity)。如果粒度太大则不利于多线程的效率，
如果粒度太小则`mutex`个数太多则容易发生死锁(deadlock)

### 死锁：现象与解决方案

如果某个操作需要同时锁住两个`mutex`，则两个线程同时进行此操作时则可能发生死锁现象。

```c++
class some_big_object;
void swap(some_big_object& lhs, some_big_object& rhs);

class X
{
private:
  some_big_object some_detail;
  std::mutex m;
public:
  X(some_big_object const& sd):some_detail(sd){}

  friend void swap(X& lhs, X& rhs)
  {
    if(&lhs==&rhs)
      return;
    std::lock(lhs.m, rhs.m);    // 同时为两个锁加锁
    std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
    std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
    swap(lhs.some_detail, rhs.some_detail);
  }
};
```

`std::lock`能够能够保证两个锁同时锁住，或者同时不锁  
`std::lock_guard`的第二个参数`std::adopt_lock`表示传入的`mutex`已经被锁，只需要转移所有权无需再锁

但是这种方法并不能避免所有的死锁情况。以下原则可以帮助避免死锁

#### 避免嵌套锁

当你为一个锁加锁时，不要再为其它锁加锁

#### 加锁时避免调用用户自定义代码

因为不确定用户自定义代码中执行了什么操作，很可能为另一个锁加锁

#### 以固定的顺序加锁

如果必须为不同的`mutex`加锁且必须分步完成，则可以在所有线程中使用固定顺序加锁

#### 使用结构化的锁

```c++
hierarchical_mutex high_level_mutex(10000);
hierarchical_mutex low_level_mutex(5000);

int do_low_level_stuff();

int low_level_func()
{
  std::lock_guard<hierarchical_mutex> lk(low_level_mutex);
  return do_low_level_stuff();
}

void high_level_stuff(int some_param);

void high_level_func()
{
  std::lock_guard<hierarchical_mutex> lk(hight_level_mutex);
  high_level_stuff(low_level_func());
}

void thread_a()
{
  high_level_func();
}

hierarchical_mutex other_mutex(100);
void do_other_stuff();

void other_stuff()
{
  high_level_func();
  do_other_stuff();
}

void thread_b()
{
  std::lock_guard<hierarchical_mutex> lk(other_mutex);
  other_stuff();
}
```

## 保护数据的其它方式
