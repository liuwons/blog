---
layout: post
title: C语言中的函数名与函数指针
date: 2015-01-18 12:06:44
tags: C
categories: code
---

函数指针是指向函数的指针变量。因而“函数指针”首先应是指针变量，只不过该指针变量指向函数。这正如用指针变量可指向整型变量、字符型、数组一样，这里是指向函数。C在编译时，每一个函数都有一个入口地址，该入口地址就是函数指针所指向的地址。有了指向函数的指针变量后，可用该指针变量调用函数，就如同用指针变量可引用其他类型变量一样，在这些概念上是大体一致的。函数指针有两个用途：调用函数和做函数的参数。
<!-- more -->

### 1.函数指针变量
函数的地址也可以像数据变量一样存储在一个函数指针变量里。然后就可以通过函数指针来调用对应的函数了。
如果有一个函数:
``` cpp
void DemoFunc(int n)
{
	printf("val:%d\n", n);
}
```
声明一个指向DemoFunc的函数指针:
``` cpp
void (*Fun)(int);   //也可写成void (*Fun)(int n);
```

### 2.用函数指针调用函数
声明一个函数指针Fun之后，可以将其赋值为DemoFunc的地址，然后通过Fun函数指针调用DemoFunc：
``` cpp
void DemoFunc(int n)
{
    printf("val:%d\n", n);
}

void main()
{
    DemoFunc(1);     	//直接调用

    void (*Fun)(int);
    Fun = &DemoFunc;  	//将DemoFunc函数的地址赋给Fun变量
    (*Fun)(20);    		//通过函数指针Fun来调用DemoFunc。
}
```
在这里DemoFunc与Fun的类型关系看着类似于int 与int* 的关系。

### 3.调用函数的其它格式
函数指针也可如下使用，来完成同样的事情：
``` cpp
void DemoFunc(int n)
{
	printf("val:%d\n", n);
}

void main()
{
	void (*Fun)(int);

	Fun = DemoFunc;  //将DemoFun函数的地址赋给Fun函数指针
	Fun(1);    		//这是通过函数指针变量调用DemoFunc函数的。
}
```
还可如下调用:
``` cpp
void main()
{
   Fun = &DemoFunc;  //将DemoFun函数的地址赋给Fun变量
   Fun(1);    		 //这是通过函数指针变量来调用DemoFunc函数的。
}
```
这样也行：
``` cpp
void main()
{
   Fun = DemoFunc;  //将DemoFun函数的地址赋给Fun变量
   (*Fun)(1);    //这是通过函数指针变量来调用DemoFunc函数的。
}
```
其实函数还可以这样调用：
``` cpp
void main()
{
	(*DemoFunc)(1);
}
```

### 4. 总结
1. DemoFunc的函数名与Fun函数指针都是一样的，即都是函数指针。DemoFunc函数名是一个函数指针常量，而Fun是一个函数数指针变量。
2. 函数名调用如果都得如(*DemoFunc)(1)；这样，书写起来很不方便。所以C语言的设计者设计成也可允许DemoFunc(1)这样调用。
3. 为统一起见，Fun函数指针变量也可以Fun(10)的形式来调用。
4. 赋值时，既可以Fun = &DemoFunc这样的形式，也可Fun = DemoFunc。
