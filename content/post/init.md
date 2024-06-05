---
title: "C++中的各种初始化方法"
date: 
lastmod: 
draft: true
categories: ["计算机基础", "C/C++"]
keywords: []
description: ""
author: "Yujie He"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""
---

ref:

[03_CS106L_Initialization_&_References_S24.pdf (stanford.edu)](https://web.stanford.edu/class/cs106l/lectures/03_CS106L_Initialization_&_References_S24.pdf)

[1.4 — Variable assignment and initialization – Learn C++ (learncpp.com)](https://www.learncpp.com/cpp-tutorial/variable-assignment-and-initialization/)

[1.6 — Uninitialized variables and undefined behavior – Learn C++ (learncpp.com)](https://www.learncpp.com/cpp-tutorial/uninitialized-variables-and-undefined-behavior/)

[C++11 统一初始化（Uniform Initialization）-CSDN博客](https://blog.csdn.net/danshiming/article/details/116273447)

# 初始化（Initialization）

> [!IMPORTANT]
>
> 几种不同的初始化方法：
>
> - 默认初始化（default initialization）
> - 直接初始化(Direct initialization)
> - 拷贝初始化(Copy initialization)
> - 统一初始化(Uniform initialization)
> - 值初始化(value initialization)

## 默认初始化（Default initialization）

在一个变量被创建的时候没有提供任何的初始值叫做默认初始化。通过默认初始化得到的变量值是不确定的，这可能造成未定义的行为。

如果直接使用一个未被初始化的进行编译，可能会报错或抛出一个警告（注意！也可能没有任何警告或者错误）。

使用未初始化的变量可能会造成以下问题：

- 每次程序运行的结果都不同；
- 程序一直都是相同的错误结果；
- 程序行为不一致；
- 程序崩溃；
- 程序在一些编译器上能够通过，但是在一些编译器上不能通过。

```c++
A a;	// 默认初始化
int a; 	// 默认初始化
```



## 直接初始化(Direct initialization)

> [!IMPORTANT]
>
> 使用圆括号进行初始化叫做直接初始化。例如下面：

```c++
int a(5);
double b(1.2);
char c('d');

int d();	// 初始值为空的时候为函数声明
int e(1.2);	// 不对类型进行检查，能够通过编译：e = 1
```

- 直接初始化是为了更`高效的初始化某些复杂的对象`。直接初始化的初始值不能为空，否则表示为函数的声明；

- `直接初始化不对类型进行检查。会进行精度截断。`



## 拷贝初始化（Copy initializatino）

当用等号对变量进行初始化的方法为拷贝初始化。

```c++
int a = 5;
double b = 1.2;
char c = 'd';

int a = 1.2; 	// a = 1, 没有类型检查
```

- 对于某些复杂的对象的初始化效率较低；
- 和直接初始化类似，拷贝初始化也没有类型检查。会进行精度截断。

## 统一初始化(Uniform initialization)

统一初始化也叫列表初始化（list initialization），C++为了统一各种初始化的不同形式，创造了统一初始化。使用花括号`{}`表示。

```c++
int a{2};						// a = 2
double b{1.2};					// b = 1.2;
int c = {2};					// c = 2;
double d = {2.2};				// d = 2.2

int v1[]{1, 2, 3};				// value = {1, 2, 3};
std::vector<int> v2{4, 5, 6};	// v2 = {4, 5, 6};
std::vector<std::string> v3{"hello", "world"};	// v3 = {"hello", "world"};

std::map<int, std::string> v4{{0, "a"}, {1, "b"}};	// v4 = {{1, "a"}, {2, "b"}};

int v5{1.2};					// error, 1.2(double) 不能赋值为int
int v6{};						// 值初始化(value initialization)见下一节
```

- `不会发生精度截断（narrowing conversions），如果初始值的类型不同那么编译过程会发生错误；`
- 形式统一，大多数情况下（除了精度截断）都首先选择统一初始化或值初始化。
- 原理：
  - 使用统一初始化，系统首先会自动调用值初始化（value initialization），将初始化值转化为std::initializer_list
  - 使用得到的std::initializer_list来初始化变量



- 对于对象的初始化来说，如果定义了参数为std::initializer_list的构造函数，优先调用该构造函数

- 例如：

```c++
#include <iostream>
#include <vector>

class Point{
public:
    Point(int _x, int _y){
        std::cout << "call Point::Point(int, int)" << std::endl;
    }

    Point(std::initializer_list<int> init){
        std::cout << "call Point::Point(std::initializer_list<int> init)" << std::endl;
    }
};

int main(){
    // uniform initialization
    Point p1(2, 3);			// call Point(int _x, int _y)
    Point p2{1, 2, 3};    	// call Point(std::initializer_list<int> init)
    Point p3{1, 2};			// call Point(std::initializer_list<int> init)
    Point p4 = {1, 2, 3, 4, 5};	// call Point(std::initializer_list<int> init)
}
```

输出：

`call Point::Point(int, int)`

`call Point::Point(std::initializer_list<int> init)`

`call Point::Point(std::initializer_list<int> init)`

`call Point::Point(std::initializer_list<int> init)`

- 当没有参数为std::initializer_list的构造函数时，调用和std::initializer_list中元素个数相同的构造函数。
- 当没有参数为std::initializer_list的构造函数时，当对应的构造函数定义为exolict时候不能使用std::initializer_list进行隐式赋值，必须进行显式调用。

```c++
#include <iostream>
#include <vector>

class Point{
public:
    Point(int _x, int _y){
        std::cout << "call Point::Point(int, int)" << std::endl;
    }

    Point(int _x, int _y, int _z){
        std::cout << "call Point::Point(int, int, int)" << std::endl;
    }
};

int main(){
    // uniform initialization
    Point p1(2, 3);			// call Point(int _x, int _y)
    Point p2{1, 2, 3};    	// call Point(int _x, int _y, int_z)
    Point p3{1, 2};			// call Point(int _x, int _y)
    Point p4 = {1, 2};	// call Point(int _x, int _y)
    Point p5 = {1, 2, 3}; 	// Error
}
```

## 值初始化（value initialization）

当花括号中的初始化值为空的时候，是值初始化，往往会直接初始化为0（此时叫做零初始化, zero initialization）或者空；

值初始化确保了即使是那些没有定义默认构造函数的类的对象，也能够被正确地初始化。

例如：

```c++
int a{};	// a = 0
float b{}; 	// b = 0
double c{}; // c = 0
char d{};	// d + 0 = 0

// 按照下面这些方式初始化类对象的时候也算是值初始化		 
A x1 = A();
A x2 = A{};
A* x3 = new A();
A* x4 = new A{};
```

- 如果类型A是数组类型，则数组的每个元素都是值初始化
- 局部静态对象在没有显式初始化时会进行值初始化。



对类进行值初始化：

- 如果需要初始化的类有用户自定义的默认构造函数，则调用默认构造函数；
- 如果需要初始化的类有编译器生成的默认构造函数，先0值初始化再调用；
- 如果没有默认构造函数则报错。



聚合类:

满足下列条件的类叫做聚合类：

- 类成员变量的访问权限都必须为public的
- 类中不能出现任何构造函数（本质上是编译器帮助用户进行了定义）
- 不能有类中初始化器
- 不能有基类或者虚函数

聚合类的设计目的：

- 一些类仅仅只是用来将多个变量进行组合在一起，如果这种情况需要用户手动编写构造函数则变得较为复杂，C++提供了聚合类的概念。聚合类能够更方便的进行初始化，而不必用户自定义繁琐的各种构造函数。



对聚合类进行值初始化相当于对类中的每个变量进行值初始化。