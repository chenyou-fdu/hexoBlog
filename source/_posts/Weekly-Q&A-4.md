title: "一周Q&A #4"
date: 2016-04-15 18:41:00
tags: [日常]
category: Weekly Q&A
---

### 在C++中如何处理unicode串？在这个过程中，char、wchar以及wstring是如何使用的？
先理解`unicode`、`UTF-8`这些概念的区别，参考[字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)。

区别`char`和`wchar_t`。`char`处理字符，大小是`1-bytes`，而`wchar_t`是处理宽字符，在Linux系统上是`4-bytes`而Windows系统上是`2-bytes`。

`std::string`是`basic_string`在`char`类型上的实例化，而`std::wstring`则是在`wchar_t`上的实例化。
可以使用`std::string`来存中文或者其他的Unicode串，只不过用多个`char`来表示一个unicode字符了，所以可以使用`std::wstring`来处理。

参考[Stack Overflow](http://stackoverflow.com/questions/402283/stdwstring-vs-stdstring)。

### Lambda函数如何递归调用？如何在Lambda函数中调用成员函数？
调用成员函数需要捕捉`this`，参考[Stack Overflow](http://stackoverflow.com/questions/11284059/call-method-inside-lambda-expression)：
```cpp
class Test {
public:
	void fun() {
		cout << "FUN" << endl;
	}
	void lambdaTest() {
		auto lambdaFun = [this]() {
			this->fun();
		};
		lambdaFun();
	}
};
```

若想在Lambda函数里进行递归，需要捕捉函数本身的引用，并且不能使用`auto`进行类型推断，参考[Stack Overflow](http://stackoverflow.com/questions/2067988/recursive-lambda-functions-in-c11)。
```cpp
//std::function类型定义在functional.h头文件里，void是指返回类型，两个int是指参数类型
function<void(int, int)> t = [&t](int a, int b)->void{
  if (a < b) t(a + 1, b);
  else cout << "DONE" << endl;
};
```
<!-- more  -->

### C++中比较函数需要遵守的Strict Weak Ordering是什么意思？
参考[Stack Overflow](http://stackoverflow.com/questions/979759/operator-and-strict-weak-ordering)，当定义自定义的比较函数时，需要满足这个要求。

也就是说，给一个比较函数`f(a,b)`，当`a == b`时，`f(a,b)`和`f(b,a)`都必须是`false`。

### CMake中如何打开C++11？如何导入Eigen库？
在CMake中需要使用C++11的特性时，参考[Stack Overflow](http://stackoverflow.com/questions/10984442/how-to-detect-c11-support-of-a-compiler-with-cmake/20165220#20165220)和[Stack Overflow](http://stackoverflow.com/questions/10851247/how-to-activate-c-11-in-cmake)，可以使用`CHECK_CXX_COMPILER_FLAG`或者`CXX_STANDARD`。

而当使用Eigen库时，需要使用`INCLUDE_DIRECTORIES `和环境变量，参考[Stack Overflow](http://stackoverflow.com/questions/12249140/find-package-eigen3-for-cmake)。

### abs(double)函数中的坑
其实在C++中，使用`std::abs`就可以满足所有的需求了，因为它重载了基本所有的类型。而在C中，`abs`只用于整数，而对于浮点数，就得使用`fabs`了。

然而这两个函数在C++中也可以使用，所以要注意`std::`限定符。参考[Stack Overflow](http://stackoverflow.com/questions/3118165/when-do-i-use-fabs-and-when-is-it-sufficient-to-use-stdabs)和[Stack Overflow](http://stackoverflow.com/questions/33738509/whats-the-difference-between-abs-vs-fabs-in-c)。

### double比较如何最优？
浮点数类型不能直接比较，一般会使用两数做差求绝对值，再和某个数比较：
```cpp
bool AreSame(double a, double b) {
    return fabs(a - b) < EPSILON;
}
```

但是这种方法其实不见得适用所有的情况，所以不同的情况要单独讨论。参考[Stack Overflow](http://stackoverflow.com/questions/17333/most-effective-way-for-float-and-double-comparison)

### 为什么函数式编程比较适用于并行？
首先理解什么是`side effect`，参考[StackExchange](http://programmers.stackexchange.com/questions/40297/what-is-a-side-effect)，意思就是说是否会改变状态。

而函数式编程是没有`side effect`的，数据是`immutable`的（没有状态，计算结果也不依赖状态），所以并行起来比较容易。参考[知乎](https://www.zhihu.com/question/20494346)和[Quora](https://www.quora.com/Why-does-functional-programming-favor-concurrency)。

### Stack Overflow是什么？为何会发生？
栈是处于地址空间的上方，然后从上往下使用，而堆则是位于下方，从下往上使用。所以栈就有可能用完地址空间，与堆发生“碰撞”。

一般造成这种情况的都是无法停止的递归，用完了栈空间。参考[StackOverflow](http://stackoverflow.com/questions/214741/what-is-a-stackoverflowerror)。

### C++在释放内存时，如何知道需要释放内存的大小？
C++中是在分配数组空间时多分配了4个字节的大小，专门保存数组的大小，在进行`delete`操作时就可以取出这个保存的数来进行释放了。

参考[Blog](http://blog.csdn.net/nodeathphoenix/article/details/39693865)、[知乎](https://www.zhihu.com/question/25556263)和[知乎](https://www.zhihu.com/question/41567197)。

### 关于堆和栈的大小。
这两个概念可以参考[Stack Overflow](http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)上这个问题的解答。

一般栈是属于一个线程的，大小也由这个线程决定（编译器决定），而堆是与这个应用挂钩的，所以大小也和这个应用分配的有关（系统决定），并且会逐步增加。

还可以参考[Stack Overflow](http://stackoverflow.com/questions/12687274/size-of-stack-and-heap-memory)。