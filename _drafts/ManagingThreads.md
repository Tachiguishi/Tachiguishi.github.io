---
layout: post
title:  Managing Threads
date:   2016-12-03 06:33:00 +0800
categories:
  - Reading
  - C++ Concurrency in Action by Anthony Williams
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

传给`std::thread`对象的可以是任何可调用的东西

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

上述调用可以使用`C++11`的`lambda`表达式简化  

```c++
std::thread my_thread([]{
  do_something();
  do_something_else();
});
```

### 线程运行方式

你必须在你的`std::thread`就像被销毁之前决定被挤启动的新线程以何种方式运行。  
如果你没有指定，在`std::thread`对象被销毁时会调用`std::terminate()`来终结你的线程

#### `join()`(主线程等待新启动的线程结束)

一个`std::thread`对象只可以执行一次`join()`  
`join()`调用后，其`joinable()`会返回`false`  
主线程会在`join()`被调用的地方等待线程结束

* exception-safety  
为了确保`join()`会被调用，可以使用Resource acquisition is initialization (RAII)策略

```c++
#include <thread>

class thread_guard
{
    std::thread& t;
public:
    // 构造函数，避免隐性转换
    explicit thread_guard(std::thread& t_):
        t(t_)
    {}
    ~thread_guard()
    {
        if(t.joinable())
        {
            t.join();
        }
    }

    // 禁止复制和赋值操作
    thread_guard(thread_guard const&)=delete;
    thread_guard& operator=(thread_guard const&)=delete;
};

void do_something(int& i)
{
    ++i;
}

struct func
{
    int& i;

    func(int& i_):i(i_){}

    void operator()()
    {
        for(unsigned j=0;j<1000000;++j)
        {
            do_something(i);
        }
    }
};

void do_something_in_current_thread(){}

void f()
{
    int some_local_state;
    func my_func(some_local_state);
    std::thread t(my_func);
    thread_guard g(t);

    do_something_in_current_thread();
}

int main()
{
    f();
}
```

#### `detach()`(主线程继续执行，新启动的线程在后台自己运行)

将线程放到后台运行必须注意的是，当主线程被销毁时后台线程可能还在继续运行，
这时如果后台线程使用了主线程中的指针或引用也会被销毁，后台线程很可能会出错。  
如果遇到这种情况可以通过将外部数据复制到线程内部而不是使用指针或引用，或者改用`join`

调用`detach()`当主线程销毁`std::thread`对象时，不会调用`std::terminate()`来线程

注意，被`detach()`的线程已经和`std::thread`对象脱离关系，所以无法再`join()`

```c++
std::thread t(do_background_work);
t.detach();
assert(!t.joinable());
```

当然在调用`detach()`时需要确定该线程与`std::thread`对象依然关联，
和`join()`时一样，可以使用`joinable()`来进行判断

## 向线程入口函数传递参数

向线程入口函数传递参数只要将参数添加到`std::thread`的构造函数中即可

```c++
void f(int i,std::string const& s);
std::thread t(f,3,"hello");
```

则被启动的线程会调用函数`f(3,"hello")`。  
注意，尽管参数传递时会把参数赋值给函数的内部变量，
但传递字符串时实际传递的是`char const*`，在线程内部再转化为`std::string`。  
所以如果传递的是指针时需要注意指针的生命周期，不过可以使用强制转换避免这个问题

```c++
void f(int i,std::string const& s);
void not_oops(int some_param)
{
char buffer[1024];
sprintf(buffer,"%i",some_param);
std::thread t(f,3,std::string(buffer));
t.detach();
}
```

## 转换线程的所有权

`std::thread`对象和`std::unique_ptr`一样，只可移动不可复制

```c++
void some_function();
void some_other_function();
std::thread t1(some_function);
std::thread t2=std::move(t1);
t1=std::thread(some_other_function);
std::thread t3;
t3=std::move(t2);
t1=std::move(t3); //此步会道次`std::terminate()`被调用，
                  //但此时还没决定线程运行方式，所以很可能会出错
```

线程对象也可以作为参数或返回值

```c++
std::thread f()
{
    void some_function();
    return std::thread(some_function);
}
std::thread g()
{
    void some_other_function(int);
    std::thread t(some_other_function,42);
    return t;
}
void f(std::thread t);
void g()
{
void some_function();
f(std::thread(some_function));
std::thread t(some_function);
f(std::move(t));
}
int main()
{
    std::thread t1=f();
    t1.join();
    std::thread t2=g();
    t2.join();
}
```

改善`thread_guard`

```c++
#include <thread>
#include <utility>

class scoped_thread
{
    std::thread t;
public:
    explicit scoped_thread(std::thread t_):
        t(std::move(t_))
    {
        if(!t.joinable())
            throw std::logic_error("No thread");
    }
    ~scoped_thread()
    {
        t.join();
    }
    scoped_thread(scoped_thread const&)=delete;
    scoped_thread& operator=(scoped_thread const&)=delete;
};

void do_something(int& i)
{
    ++i;
}

struct func
{
    int& i;

    func(int& i_):i(i_){}

    void operator()()
    {
        for(unsigned j=0;j<1000000;++j)
        {
            do_something(i);
        }
    }
};

void do_something_in_current_thread()
{}

void f()
{
    int some_local_state;
    scoped_thread t(std::thread(func(some_local_state)));

    do_something_in_current_thread();
}

int main()
{
    f();
}
```

```c++
void do_work(unsigned id);
void f()
{
std::vector<std::thread> threads;
for(unsigned i=0;i<20;++i)
{
threads.push_back(std::thread(do_work,i));
}
std::for_each(threads.begin(),threads.end(),
std::mem_fn(&std::thread::join));
}
```

## 动态决定运行的线程数量

`std::thread::hardware_concurrency()`可以获取能真实并行的线程数量  
但也可能无法获取机器中的此信息返回`0`

并行算法  

```c++
#include <thread>
#include <numeric>
#include <algorithm>
#include <functional>
#include <vector>
#include <iostream>

template<typename Iterator,typename T>
struct accumulate_block
{
    void operator()(Iterator first,Iterator last,T& result)
    {
        result=std::accumulate(first,last,result);
    }
};

template<typename Iterator,typename T>
T parallel_accumulate(Iterator first,Iterator last,T init)
{
    unsigned long const length=std::distance(first,last);

    if(!length)
        return init;

    unsigned long const min_per_thread=25;
    unsigned long const max_threads=
        (length+min_per_thread-1)/min_per_thread;

    unsigned long const hardware_threads=
        std::thread::hardware_concurrency();

    unsigned long const num_threads=
        std::min(hardware_threads!=0?hardware_threads:2,max_threads);

    unsigned long const block_size=length/num_threads;

    std::vector<T> results(num_threads);
    std::vector<std::thread>  threads(num_threads-1);

    Iterator block_start=first;
    for(unsigned long i=0;i<(num_threads-1);++i)
    {
        Iterator block_end=block_start;
        std::advance(block_end,block_size);
        threads[i]=std::thread(
            accumulate_block<Iterator,T>(),
            block_start,block_end,std::ref(results[i]));
        block_start=block_end;
    }
    accumulate_block<Iterator,T>()(block_start,last,results[num_threads-1]);

    std::for_each(threads.begin(),threads.end(),
        std::mem_fn(&std::thread::join));

    return std::accumulate(results.begin(),results.end(),init);
}

int main()
{
    std::vector<int> vi;
    for(int i=0;i<10;++i)
    {
        vi.push_back(10);
    }
    int sum=parallel_accumulate(vi.begin(),vi.end(),5);
    std::cout<<"sum="<<sum<<std::endl;
}
```

## 识别线程

`std::thread::id`类型的一个数值标识线程，它可以进行比较操作  
可以通过`get_id()`来获取，通过`std::this_thread::get_id()`来获取当前线程  

如果`std::thread`对象没有关联的线程，调用`get_id()`会返回一个默认值
