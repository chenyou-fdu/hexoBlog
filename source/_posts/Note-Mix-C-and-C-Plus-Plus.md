title: "#开发笔记# 如何混编C++和C"
date: 2015-07-19 16:34:24
tags: [开发，笔记, C++]
category: 开发笔记
---

## 在C++中调用C函数
在C++中调用C函数比较简单，因为C++基本支持C中的语法。

首先在一个头文件里声明一个C函数，注意这里函数前面需要加上一个`extern`，原因可以参考引用网站：
```c
//cHeader.h
#ifndef C_HEADER
#define C_HEADER

extern void testFun();

#endif
```

然后进行实现：
```c
//cSource.c
#include <stdio.h>
#include "cHeader.h"
void testFun() {
    printf("Hello World");
}
```

接下来需要在C++代码中进行调用，注意引用头文件时需要放在`extern`块中：
```cpp
//cppSource.cpp
extern "C" {
    #include "cHeader.h"
}
 
int main(int argc,char*argv) {
    testFun();
    return 0;
}
```
<!-- more  -->

## 在C中调用C++函数
而在C语言中调用C++代码，就比较复杂了，因为涉及到类和模版等C语言不支持的特性。

### 不涉及类的函数
首先考虑不涉及类的函数。在C++头文件中的声明如下，注意函数前的`extern "C"`：
```cpp
//cppHeader.h
#ifndef CPP_HEADER
#define CPP_HEADER

extern "C" void testFunc();

#endif
```
然后在C++源文件中进行实现：
```cpp
//cppSource.cpp
#include "cppHeader.h"
#include <iostream>
using namespace std;

void testFunc() {
    cout << "Hello World" << endl;
}
```

而在C语言中对这个函数进行调用则需要声明这个函数：
```c
extern void testFunc();
int main(int argc,char*argv) {
    testFun();
    return 0;
}
```
### 涉及类的函数 
当C++代码中涉及到了面向对象的内容的话，就需要使用空指针来完成混合编程了。
首先在头文件中声明一个类和相应的成员函数：
```cpp
//cppHeader.h
#ifndef CPP_HEADER
#define CPP_HEADER
class TestObj {
    public:
        TestObj(int data) : classData(data) {}
        inline int getData()  {
            return classData;
        }
    private:
        int classData
};
```

如果我们需要在C中使用这个类，那么还需要一些Wrap函数，首先定义一个C++头文件：
```cpp
//cppWrapHeader.h
#ifndef CPP_WRAP_HEADER
#define CPP_WRAP_HEADER
#include "cppHeader.h"
extern "C" voidTestObj_new(int data);
extern "C" void TestObj_free(voidtestObj);
extern "C" int TestObj_getData(voidtestObj);
#endif
```

然后实现这些Wrap函数，在函数中对`void指针`进行类型转换，使其能满足使用的要求：
```cpp
//cppWrapSource.cpp
#include "cppWrapHeader.cpp"
extern "C" voidTestObj_new(int data) {
    return static_cast<void*>(new TestObj(data));
}
extern "C" void TestObj_free(voidtestObj) {
    delete static_cast<TestObj*>(testObj);
}
extern "C" int TestObj_getData(voidtestObj) {
    return static_cast<TestObj*>(testObj)->getData();
}
```

接下来当我们需要在C语言中使用这些函数时，只需要操作`void指针`就行了：
```c
//cSource.c
#include "cppWrapHeader.h"
int main(int argc,char*argv) {
    voidtestObj = TestObj_new(10);
    int result = TestObj_getData(testObj);
    TestObj_free(testObj);
    return 0;
}
```
## CMAKE中的设定
完成混编时，在CMAKE文件中不需要过多的设置只用设置好两个编译器就行了：
```cmake
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
```

## 参考
[Calling “C++” class member function from “C” code](http://stackoverflow.com/questions/3583353/calling-c-class-member-function-from-c-code)
[How to call a C++ method from C?](http://stackoverflow.com/questions/14815274/how-to-call-a-c-method-from-c)
[C++项目中的extern "C" {}](http://www.cnblogs.com/skynet/archive/2010/07/10/1774964.html)