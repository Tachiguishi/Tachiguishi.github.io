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

```c++
std::mutex mut;
std::queue<data_chunk> data_queue;
std::condition_variable data_cond;

void data_preparation_thread()
{
  while(more_data_to_prepare())
  {
    data_chunk const data=prepare_data();
    std::lock_guard<std::mutex> lk(mut);
    data_queue.push(data);
    data_cond.notify_one();
  }
}

void data_processing_thread()
{
  while(true)
  {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk, []{return !data_queue.empty()})
    data_chunk data=data_queue.front();
    data_queue.pop();
    lk.unlock();    // 数据处理可能很毫时，提前解锁
    process(data);
    if(is_last_chunk(data))
      break;
  }
}
```

当处理数据的线程处理完数据后，将数据写入共享数据`data_queue`中，
然后调用`std::condition_variable`的`notify_one()`函数通知其它线程数据已经准备好  
在等待线程中调用`std::condition_variable`的`wait()`首先校验作为第二个参数的`lambda`函数
是否符合条件。  
如果符合(`return true`)，则执行之后的语句；
如果不符合(`return false`)，则先解锁，然后进入等待状态，
直到`notify_one()`函数被调用时再次校验`lambda`函数是否符合条件。

`wait()`在每次校验条件时都会先加锁，校验失败则解锁。所以需要使用`std::unique_lock`而非`std::lock_guard`

### building a thread-safe queue with condition variables

```c++
#include <queue>
#include <memory>
#include <mutex>
#include <condition_variable>

template<typename T>
class threadsafe_queue
{
private:
  mutable std::mutex mut;
  std::queue<T> data_queue;
  std::condition_variable data_cond;
public:
  threadsafe_queue(){}
  threadsafe_queue(threadsafe_queue const& other)
  {
    std::lock_guard<std::mutex> lk(othre.mut);
    data_queue=other.data_queue;
  }

  void push(T new_value)
  {
    std::lock_guard<std::mutex> lk(mut);
    data_queue.push(new_value);
    data_cond.notify_one();
  }

  void wait_and_pop(T& value)
  {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk, [this]{return !data_queue.empty();});
    value=data_queue.front();
    data_queue.pop();
  }

  std::shared_ptr<T> wait_and_pop()
  {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk, [this]{return !data_queue.empty();});
    std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
    data_queue.pop();
    return res;
  }

  bool try_pop(T& value)
  {
    std::lock_guard<std::mutex> lk(mut);
    if(data_queue.empty())
      return false;
    value=data_queue.front();
    data_queue.pop();
    return true;
  }

  std::shared_ptr<T> try_pop()
  {
    std::lock_guard<std::mutex> lk(mut);
    if(data_queue.empty())
      return std::shared_ptr<T>();
    std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
    data_queue.pop();
    return res;
  }

  bool empty() const
  {
    std::lock_guard<std::mutex> lk(mut);
    return data_queue.empty();
  }
};

threadsafe_queue<data_chunk> data_queue;

void data_preparation_thread()
{
  while(more_data_to_prepare())
  {
    data_chunk const data=prepare_data();
    data_queue.push(data);
  }
}

void data_processing_thread()
{
  while(true)
  {
    data_chunk data;
    data_queue.wait_and_pop(data);
    process(data);
    if(is_last_chunk(data))
      break;
  }
}
```

如果同时有多个线程在等待某一个线程中的事件，在调用`notify_one()`时，一次只会有一个线程从等待中被唤醒。
如果想同时唤醒所有的线程，可以使用`notify_all()`

## waiting for one-off events with futures

`std::future<>`和`std::shared_future<>`包含在`<future>`头文件中

### returning values from background tasks

```c++
#include <future>
#include <iostream>

int find_the_answer_to_ltuae();
void do_other_stuff();
int main()
{
  std::future<int> the_answer=std::async(find_the_answer_to_ltuae);
  do_other_stuff();
  std::std::cout << "The answer is" << the_answer.get() << std::endl;
}
```

和`std::thread`一样，传入`std::async`的函数也可以附带参数

```c++
#include <string>
#include <future>

struct X
{
  void foo(int, std;;string const&);
  std::string bar(std::string const&);
};
X x;
auto f1=std::async(&X::foo, &x, 42, "hello");   // p->foo(42, "hello"), p is &x
auto f2=std::async(&X::bar, x, "goodbye");      // temp.bar("goodbye"),
                                                // temp is a copy of x

struct Y
{
  double operator()(double);
};
Y y;
auto f3 = std::async(Y(), 3.14);// temp(3.14), temp is move-constructed from Y()
auto f4 = std::async(std::ref(y), 2,718); // y(2.718)

X baz(X&);
std::asyc(baz, std;;ref(x));  // baz(x)
class move_only
{
public:
  move_only();
  move_only(move_only&&);
  move_only(move_only const&) = delete;
  move_only& operator=(move_only&&);
  move_only& operator=(move_only const&) = delete;

  void operator()();
};
// temp(), temp is constructed form std::move(move_only())
auto f5 = std::async(move_only());
```

`std::async`不一定会启动一个新线程，但可以用参数手动控制

```c++
auto f6=std::async(std::launch::async, Y(), 1.2); // run in new thread
auto f7=std::async(std::launch::deferred, std::ref(x));// run in wait() or get()
auto f8=std::async(std::launch::deferred | std::launch::async,
                   baz, std::ref(x)); // implementation chooses
auto f9=std::async(baz, std::ref(x)); // implementation chooses
f7.wait();  // invoke deferred function
```

### associating a task with a future

`std::packaged_task<>`接受函数签名作为模版参数，如`std::packaged_task<double(double)>`
函数签名中的返回值类型即为调用`get_future()`时的返回的`std::future<>`类型

```c++
#include <deque>
#include <mutex>
#include <future>
#include <thread>
#include <utility>

std::mutex m;
std::deque<std::packaged_task<void()>> tasks;

bool gui_shutdown_message_received();
void get_and_process_gui_message();

void gui_thread()
{
  while(!gui_shutdown_message_received())
  {
    get_and_process_gui_message();
    std::packaged_task<void()> task;
    {
      std::lock_guard<std::mutex> lk(m);
      if(tasks.empty())
        continue;
      task=std::move(tasks.front());
      tasks.opp_front();
    }
    task();
  }
}

std:: thread gui_bg_thread(gui_thread);

template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
  std::packaged_task<void()> task(f);
  std::future<void> res=task.get_future();
  std::lock_guard<std::mutex> lk(m);
  tasks.push_back(std::move(task));
  return res;
}
```

### making (std::)promises

在单个线程中处理多个通信连接

```c++
#include <future>

void process_connections(connection_set& connections)
{
  while(!done(connections))
  {
    for(connection_iterator
          connection=connections.begin(), end=connections.end();
        connection!=end;
        ++connection)
    {
      if(connection->has_incoming_data())
      {
        data_packet data=connection->incoming();
        std::promise<payload_type>& p=connection->get_promise(data.id);
        p.set_value(data.payload);
      }
      if(connection->has_outgoing_data())
      {
        outgoing_packet data=connection->top_of_outgoing_queue();
        connection->send(data.plyload);
        data.promise.set_value(true);
      }
    }
  }
}
```
