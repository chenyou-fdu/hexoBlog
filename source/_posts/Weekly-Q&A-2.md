title: "一周Q&A #2"
date: 2015-07-24 22:19:45
tags: [日常]
category: Weekly Q&A
---
一直偷懒没写，其实问题早就整理好了。

### Linux下的编码和DOS下的编码有什么不同？
DOS下的文本文件以`\r\n`（十六进制为`0x0D0A`）作为换行标志，表示成十六进制就是0D 0A。而Linux下的文本文件是以`\n`（十六进制为`0x0A`）作为换行标志的。这样在Linux下使用DOS下的文档（或者在DOS中使用Linux文档），都会出现很麻烦的错误，这时就需要使用Dos2Unix和Unix2Dos这两个小工具来进行转换。

### CMAKE中如何连接pthread和math库？
假设项目名称是Project，那么需要加上：

```cmake
target_link_libraries(WapitiMarisa_Final m pthread)
```

### C++/C中变量内存分配的原理？
分为栈、堆、自由存储区、常量存储区以及全局/静态存储区。具体可以参考[C++内存分配方式详解](http://www.cnblogs.com/daocaoren/archive/2011/06/29/2092957.html)这篇博文。
<!-- more  -->

### Python中字符串的join()函数有什么用处？
Python中使用join()函数连接字符串

```Python
'1'.join('abc')
# 输出为a1b1c
arr = ['What','the','hell']
' '.join(arr)
# 输出为What the hell
```
可以参考SO上的[Explain Python .join()](http://stackoverflow.com/questions/1876191/explain-python-join)

### Python中为何要使用with as结构？
为了方便进行文件处理中`Exception`的处理，可以参考这篇博文[理解Python中的with…as…语法](http://zhoutall.com/archives/325)

### 理解Python中文编码？
Python 2中，字符串分为str和unicode两种，而Python 3后，全部统一为unicode，这样就方便了不少。

一般为了方便处理编码问题，统一把输入的中文文本解码为unicode（用codecs库或者简单的decode），在python内部的操作都用unicode进行，然后输出时统一输出成为`utf-8`。

具体可以参考[PYTHON-进阶-编码处理小结](http://wklken.me/posts/2013/08/31/python-extra-coding-intro.html)
另外检测编码可以使用chardet这个库

### XML中有哪些字符是无效的？
简单来说，有`& < > " '`这几个，需要分别进行替换为：`&amp;`、`&lt;`、`&gt;`、`&apos;`和`&quot;`
具体可以参考[Invalid Characters in XML](http://stackoverflow.com/questions/730133/invalid-characters-in-xml)
之前在处理类XML文件时，写过几个Regex：

```Python
temp = re.sub(r'&', '&amp;', partXML)
temp = re.sub(r"'", '&apos;', temp)
temp = re.sub(r'"', '&quot;', temp)
temp = re.sub(r'(<LD_.*>.*)(<)(.*)(>)(.*</LD_.*>)', r'\g<1>&lt;\g<3>&gt;\g<5>', temp)
# <>这两个符号很难处理，要用group分开针对不同tag处理
```

### PIP安装时，出现Unable to find vcvarsall.bat怎么办？
参考这个[error: Unable to find vcvarsall.bat](http://stackoverflow.com/questions/2817869/error-unable-to-find-vcvarsall-bat)

### 'new operator'和'operator new'有什么区别？
`operator new`是用来分配原始内存的，和`malloc()`差不多，而`new operator`则是平时用来分配内存的操作符。

参考[Difference between 'new operator' and 'operator new'?](http://stackoverflow.com/questions/1885849/difference-between-new-operator-and-operator-new)。

更多的还有`placement new`、`operator delete`等，可以参考[深入探究C++的new/delete操作符](http://kelvinh.github.io/blog/2014/04/19/research-on-operator-new-and-delete/)。

### 使用Python的`ujson`库时，如何保证dumps的不是一个编码串而是编码过的文字？
设置`ensure_ascii=False`

```Python
ujson.dumps(toDumpDict, ensure_ascii=False)
```
