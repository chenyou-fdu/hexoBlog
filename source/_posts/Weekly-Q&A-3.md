title: "一周Q&A #3"
date: 2015-12-31 10:47:15
tags: [笔记, 日常]
category: Weekly Q&A
---

这个好几个月没写了，这次补上一点。

### C++中如何对std::string进行trim？
`trim`这个函数很方便，可以把字符串首尾的空格去掉，但是STL库中没有提供相关的实现。参考SO上[What's the best way to trim std::string](http://stackoverflow.com/questions/216823/whats-the-best-way-to-trim-stdstring)

这个提问，有一些自己的实现，也有说用Boost库的实现，都挺方便的。

### 在Python中，调用一个子进程来运行某一个命令，如何设置相关的环境变量？
`subprocess.Popen()`函数提供了一个参数`env`，直接传入一个保存环境变量设置的`dict`就行了。

```python
new_env = os.environ.copy()
new_env['MEGAVARIABLE'] = 'MEGAVALUE'
subprocess.Popen('path', env=new_env)
```

### 编译器报 error C4146: 一元负运算符应用于无符号类型，结果仍为无符号类型 错是什么情况？
当定义一个变量`int n = -2147483648`就会报这个错，因为编译器编译时会认为`2147483648`这个大于了`INT_MAX`，所以用`unsigned int`

来存，然后发现前面有负号，所以会报错。而有些编译器不会报错，只会warnning，这样容易导致错误。正确的用法还是用`int n = INT_MIN`。

参考[error C4146: 一元负运算符应用于无符号类型，结果仍为无符号类型](http://www.hankcs.com/program/cpp/error-c4146-%E4%B8%80%E5%85%83%E8%B4%9F%E8%BF%90%E7%AE%97%E7%AC%A6%E5%BA%94%E7%94%A8%E4%BA%8E%E6%97%A0%E7%AC%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B%EF%BC%8C%E7%BB%93%E6%9E%9C%E4%BB%8D%E4%B8%BA%E6%97%A0.html)。
<!-- more  -->

### 在使用`string.c_str()`函数得到的C风格字符串后，需要手动释放空间么？
不需要，在`string`对象被回收时，这个C字符串也会自动被回收。参考SO上这个提问[string.c_str() deallocation necessary?](http://stackoverflow.com/questions/8843604/string-c-str-deallocation-necessary)。

### VS中配置第三方库的有哪些经验？
一般来说 `inclue`和`lib`文件最好用环境变量配置（设置一个环境变量指向存放这些文件的位置），如果放在工程内部，移动工程位置时，绝对路径发生改变，需要重新配置。

而`DLL`要么和代码放在一起，要么把存放了`DLL`的文件夹路径添加到`Path`这个系统变量中去。

### C++类的静态常量成员变量如何初始化？
不能在类的内部初始化，类内只进行声明，需要在类外部进行定义。参考SO中这个问题[C++ Static Const Member Variable Usage](http://stackoverflow.com/questions/17056882/c-static-const-member-variable-usage)。

### STL库中的优先队列（priority_queue）如何使用自定义的比较函数？
因为`priority_queue`的把比较函数当作一个模板参数，所以不能直接传一个Lambda函数进去（因为这是一个简单的Object）。所以需要使用一个`decltype`操作。
```cpp
auto comp = [](const std::pair<int, int>& a, const std::pair<int, int>& b) { return a.second > b.second; };
//这里还要把comp当作参数传入才行
std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, decltype(comp)> pQueue(comp);
```

参考SO上[C++ priority_queue with lambda comparator error](http://stackoverflow.com/questions/5807735/c-priority-queue-with-lambda-comparator-error)和[Priority Queue Comparison](http://stackoverflow.com/questions/20826078/priority-queue-comparison)这两题。

### 如何向STL的vector后插入整个数组结构？
使用insert函数，例如：
```cpp
vector<int> a = {1, 2, 3};
vector<int> b = {4, 5, 6};
insert(a.end(), b.begin(), b.end());
```
但是这样好像效率不是很高，没有实际比较过
