title: "一周Q&A #1"
date: 2015-06-05 22:19:45
tags: [笔记, 日常]
category: Weekly Q&A
---

以后坚持每一周或两周记录下这段时间内的一些小的问题与其解决方法，包括开发和读论文中的各种问题。

### 在CMake设置中，如何设置Debug模式？
使用`-DCMAKE_BUILD_TYPE=`这个参数进行设置，例如下面分别就是两个模式。

```
cmake -DCMAKE_BUILD_TYPE=Debug <path>
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo <path>
```
参考SO上这个问题： [How do you set GDB debug flag with cmake?](http://stackoverflow.com/questions/10005982/how-do-you-set-gdb-debug-flag-with-cmake)

### C99中PRIu64这一类宏如何使用？
例子，注意这个头文件和这个宏定义都不可缺少：

```cpp
#define __STDC_FORMAT_MACROS
#include <inttypes.h>
#include <stdio.h>

int main() {
    uint64_t ui64 = 90;
    printf("test uint64_t : %" PRIu64 "\n", ui64);
    return 0;
}
```
可以用`%llu`、`%u`来进行代替。
参考SO上[how to printf uint64_t?](http://stackoverflow.com/questions/8132399/how-to-printf-uint64-t)和[Why doesn't PRIu64 work in this code?](http://stackoverflow.com/questions/14535556/why-doesnt-priu64-work-in-this-code)。

### 如果一个字符串不是以NULL（\0）结尾怎么进行输出？
使用`printf("%.*s", stringLength, pointerToString)`就行。
<!-- more  -->

### printf中%a是什么？
是C99标准中新定义的一种格式，用十六进制格式表示一个浮点数，相应的还有`%la`。与科学计数法`%e`相对应。

具体可以参考[Hexadecimal Floating-Point Constants](http://www.exploringbinary.com/hexadecimal-floating-point-constants/)和SO上[The format specifier %a for printf() in C](http://stackoverflow.com/questions/4826842/the-format-specifier-a-for-printf-in-c)。

<!--more-->

### 如何把一个一维数组转成多维数组来使用？
代码（C++）如下，这样就可以使用`ptr`这个数组指针以三维数组形式来进行遍历了。

```cpp
int *arr = new int[125];
int (*ptr)[5][5][5] = (int(*)[5][5][5])arr;
```
C语言中是这样，要稍微简单些；

```cpp
int *arr = new int[125];
int (*ptr)[5][5][5] = (void *)arr;
```

### C++中两个冒号代表了什么？
`::`表示最外层的命名空间，也就是global namespace。
具体参考SO上这个问题[What is the meaning of prepended double colon “::” to class name?](http://stackoverflow.com/questions/4269034/what-is-the-meaning-of-prepended-double-colon-to-class-name)。

### intptr_t这个类型是干什么的？
这个类型专门用来表示指针的地址，可以在指针对象和`intptr`对象之间进行转换。
参考这篇博文[C语言指针转换为intptr_t类型](http://www.cnblogs.com/anker/p/3438480.html)。

### C/C++中struct类如何进行初始化？
一般分成3种，顺序、C风格乱序和C++风格乱序：

```cpp
struct User {
    int id;             //id
    char name[100];     //user name
    char *home;         //home directory
    int passwd;         //password
};

/*
 * 顺序进行初始化
 */
struct User oneUser = {10, "Lucy", "/home/Lucy", 0};

/*
 * C风格乱序进行初始化
 */
struct User oneUser = {
    .name = "Lucy", 
    .id = 10,
    .home = "/home/Lucy"
};

/*
 * C风格乱序进行初始化
 */
struct User oneUser = {
    name:"Lucy", 
    id:10,
    home:"/home/Lucy"
};
```
注意在g++编译器中，乱序只能用在`*.c`的文件中，而`*.cc`和`*.cpp`都是不支持的，会报下面这个错。
`sorry, unimplemented: non-trivial designated initializers not supported`
具体可参考这篇博文[C/C++ struct初始化/复制/内存分配技巧](http://blog.csdn.net/johnny710vip/article/details/7281131)。

### 在函数内给指针分配内存空间会怎样？
在函数内给指针分配内存空间会造成拷贝，造成内存泄露而且函数外无法使用，所以要避免。

### 如何区别C++中的几种类型转换操作符？
一般有`static_cast<>()`，`reinterpret_cast<>()`，`const_cast<>()`，和`dynamic_cast<>()`这四种不同的类型转换方法。

具体的区别和使用，参考这两篇SO提问[In C++, why use static_cast<int>(x) instead of (int)x?](http://stackoverflow.com/questions/103512/in-c-why-use-static-castintx-instead-of-intx)，[When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?](http://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used)以及一篇博文[C++中static_cast, dynamic_cast, const_cast用法/使用情况及区别解析](http://blog.csdn.net/bzhxuexi/article/details/17021559)。

注意几个点：`const_cast<>()`有一个未定义行为，就是把指向常量变量的常量指针转为普通指针，这是未定义的危险行为。

```cpp
const int num = 10;
const int* numPtr = &num;
int* unsafePtr = const_cast<int*>(numPtr);
```
`dynamic_cast<>()`进行downcast时，要求基类得有一个虚函数，否则会报错。
