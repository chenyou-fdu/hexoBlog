title: "再造一个线程池"
date: 2016-12-19 22:01:18
tags: [开发, 多线程, C++]
category: 开发
---
之前写实验室项目用过一些开源的C++线程池来加速算法，然后就想着自己重新造个轮子，学习一个。
这个东西比较简单，代码也很少，已经基本写完了，放在了[Github](https://github.com/chenyou-fdu/RickThreadPool)上。但是在写的过程中，碰到了很多C++11的新特性，学到了很多东西，在这里总结一下。

## Prefect Forward 完美转发
因为要去实现一个库，所以肯定要用上template的内容，于是就碰到了这个问题。
找了下资料，通过[移动语义（move semantic）和完美转发（perfect forward](https://codinfox.github.io/dev/2014/06/03/move-semantic-perfect-forward/)这个学习了右值、右值引用、移动语义还有完美转发这一大堆东西，讲得还是很易懂的。

所以实现往线程库里添加工作函数的功能时，就用上了这个。
```cpp
template<class F, class... Args>
inline auto enqueue(F&& f, Args&&... args) -> std::future<decltype(f(args...))> 
{
    /* ... */
}
```
这里的`F&& f`就是个通用引用了，为了保证完美转发，后面代码里用上了`std::forward<T>()`函数（不这样转发到下一个函数里就会被当成左值）：
```cpp
auto task = std::make_shared< std::packaged_task<fReturnType()> >(
                std::bind(std::forward<F>(f), std::forward<Args>(args)...)
            );
```
这样，就可以在不改变语义的情况下，正确的使用左值和右值，提高效率。

<!-- more  -->

## std::bind()是什么？
`std::bind()`这个特性是用来进行偏函数应用（partial application）的，可以把参数和函数互相进行绑定。还是之前那个代码：
```cpp
std::bind(std::forward<F>(f), std::forward<Args>(args)...)
```
就把`args...`参数绑定到了函数`f`上面，这样就形成了一个新的偏函数应用`f(args...)`，因为要异步返回这个工作函数的结果，所以用了`std::packaged_task`。

## std::future和std::packaged_task
这几个都是C++11里thread库中的概念，刚学时有点难懂。
首先是`std::future`，它可用于异步任务中获取任务结果。例如使用`std::async()`来创建一个异步任务（后台会**马上**开启一个线程来执行工作函数），返回的结果就是一个`std::future`。主线程可以之后来通过这个`std::future`对象获取刚才异步任务的返回结果。
而需要更灵活执行异步任务时，就需要用到`std::packaged_task`了。这个类似一个异步版的`std::function`，可以在**需要**时才开始执行（区别使用`std::async()`函数）。
不过要注意得是，`std::function`是可以拷贝构造的，而`std::packaged_task`则不行，所以两者不能相互转换。
```cpp
template<class F, class... Args>
inline auto enqueue(F&& f, Args&&... args) -> std::future<decltype(f(args...))>
{
    typedef decltype(f(args...)) fReturnType;
    auto task = std::make_shared< std::packaged_task<fReturnType()> >(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
    std::future<fReturnType> res = task->get_future();
    taskQueue.push([task]() { (*task)(); });
    /* ... */
}
```
接着之前那个代码，这里的`task`就是一个指向`std::packaged_task<fReturnType()>`的智能指针，然后会把这个`task`再包装成一个`std::function`放入`taskQueue`中。

## Mutex和RAII
thread库里的互斥量都用到了RAII（Resource Acquisition Is Initialization），当创建Mutex对象时获得锁，而对象被释放时失去锁。
```cpp
threads.push_back(std::thread([this] {
    while (true) {
        std::function<void()> task;
        {
            std::unique_lock<std::mutex> lock(this->tMutex);
            this->tCv.wait(lock, [this] { 
                return this->stopFlag || !this->taskQueue.empty();
            });
            if (stopFlag && this->taskQueue.empty())
                return;
        }
        this->taskQueue.tryPop(task);
        task();
    }
}));
```
这段代码里，`std::unique_lock`对象离开`{}`标出的作用域就会被释放，也就是释放锁。

## 总结
写这个主要学习到的还是C++11的一些新特性，还有一些多线程开发的基础知识。后面这个代码可能还要更新，这篇文章也会随着接着写吧。

### 其他参考
* [Advantages of using forward](http://stackoverflow.com/questions/3582001/advantages-of-using-forward)
* [C++11中的std::bind](http://www.jellythink.com/archives/773)
* [Sign up
std::function and std::bind: what are they & when they should be used?](http://stackoverflow.com/questions/6610046/stdfunction-and-stdbind-what-are-they-when-they-should-be-used)
* [What is the difference between packaged_task and async](http://stackoverflow.com/questions/18143661/what-is-the-difference-between-packaged-task-and-async)