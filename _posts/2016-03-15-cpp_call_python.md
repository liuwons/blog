---
layout: post
title: 浅析 C++ 调用 Python 模块
date: 2016-03-15 00:23:55
tags: Python
categories: code
---

作为一种胶水语言，**Python** 能够很容易地调用 **C** 、 **C++** 等语言，也能够通过其他语言调用 **Python** 的模块。

**Python** 提供了 **C++** 库，使得开发者能很方便地从 **C++** 程序中调用 **Python** 模块。

具体的文档参考官方指南：
[Embedding Python in Another Application](https://docs.python.org/2/extending/embedding.html)

## 调用方法

### 1 链接到 **Python** 调用库

**Python** 安装目录下已经包含头文件( `include` 目录)和库文件 ( *Windows* 下为 `python27.lib`)。

使用之前需要链接到此库。

### 2 直接调用 **Python** 语句

```cpp
#include "python/Python.h"

int main()
{
    Py_Initialize();    ## 初始化

    PyRun_SimpleString("print 'hello'");

    Py_Finalize();      ## 释放资源
}

```

### 3 加载 **Python** 模块并调用函数

`~/test` 目录下含有 `test.py` :

```python
def test_add(a, b):
    print 'add ', a, ' and ', b
    return a+b
```

则可以通过以下代码调用 `test_add` 函数 :

```cpp
#include "python/Python.h"
#include <iostream>
using namespace std;

int main()
{
    Py_Initialize();    // 初始化

    // 将Python工作路径切换到待调用模块所在目录，一定要保证路径名的正确性
    string path = "~/test";
    string chdir_cmd = string("sys.path.append(\"") + path + "\")";
    const char* cstr_cmd = chdir_cmd.c_str();
    PyRun_SimpleString("import sys");
    PyRun_SimpleString(cstr_cmd);

    // 加载模块
    PyObject* moduleName = PyString_FromString("test"); //模块名，不是文件名
	PyObject* pModule = PyImport_Import(moduleName);
    if (!pModule) // 加载模块失败
	{
		cout << "[ERROR] Python get module failed." << endl;
		return 0;
	}
	cout << "[INFO] Python get module succeed." << endl;

    // 加载函数
    PyObject* pv = PyObject_GetAttrString(pModule, "test_add");
	if (!pv || !PyCallable_Check(pv))  // 验证是否加载成功
	{
		cout << "[ERROR] Can't find funftion (test_add)" << endl;
		return 0;
	}
	cout << "[INFO] Get function (test_add) succeed." << endl;

    // 设置参数
	PyObject* args = PyTuple_New(2);   // 2个参数
	PyObject* arg1 = PyInt_FromLong(4);    // 参数一设为4
	PyObject* arg2 = PyInt_FromLong(3);    // 参数二设为3
	PyTuple_SetItem(args, 0, arg1);
	PyTuple_SetItem(args, 1, arg2);

    // 调用函数
	PyObject* pRet = PyObject_CallObject(pv, args);

    // 获取参数
    if (pRet)  // 验证是否调用成功
	{
		long result = PyInt_AsLong(pRet);
		cout << "result:" << result;
	}

    Py_Finalize();      ## 释放资源

    return 0;
}

```

## 参数传递

### 1 **C++** 向 **Python** 传递参数

**Python** 的参数实际上是元组，因此传参实际上就是构造一个合适的元组。

常用的有两种方法：

- 使用 `PyTuple_New` 创建元组， `PyTuple_SetItem` 设置元组值

    ```cpp
    PyObject* args = PyTuple_New(3);
    PyObject* arg1 = Py_BuildValue("i", 100); // 整数参数
    PyObject* arg2 = Py_BuildValue("f", 3.14); // 浮点数参数
    PyObject* arg3 = Py_BuildValue("s", "hello"); // 字符串参数
    PyTuple_SetItem(args, 0, arg1);
    PyTuple_SetItem(args, 1, arg2);
    PyTuple_SetItem(args, 2, arg3);
    ```

- 直接使用Py_BuildValue构造元组

    ```cpp
    PyObject* args = Py_BuildValue("(ifs)", 100, 3.14, "hello");
    PyObject* args = Py_BuildValue("()"); // 无参函数
    ```
`i`, `s`, `f`之类的格式字符串可以参考 [格式字符串][1]

### 2 转换 **Python** 返回值

调用 **Python** 得到的都是PyObject对象，因此需要使用 **Python** 提供的库里面的一些函数将返回值转换为 **C++** , 例如 `PyInt_AsLong`，`PyFloat_AsDouble`， `PyString_AsString` 等。

还可以使用 `PyArg_ParseTuple` 函数来将返回值作为元组解析。

`PyArg_Parse` 也是一个使用很方便的转换函数。

`PyArg_ParseTuple` 和 `PyArg_Parse` 都使用 [格式字符串][1]

## 注意事项

1. 需要将 **Python** 的工作目录切换到模块所在路径
2. 按照模块名加载而不是文件名
3. 模块加载或者函数加载需要验证是否成功，否则可能会引起堆栈错误导致程序崩溃
4. 需要使用 `Py_DECREF(PyObject*)` 来解除对象的引用(以便Python垃圾回收)


[1]: https://docs.python.org/release/1.5.2p2/ext/parseTuple.html
