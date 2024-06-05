---
title: "C++中的左值和右值"
date: 
lastmod: 
draft: true
categories: ["C/C++"]
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

# C++中的左值和右值

[toc]

ref :

[C++中左值和右值的理解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/240833006)

[Understanding the meaning of lvalues and rvalues in C++ - Internal Pointers](https://www.internalpointers.com/post/understanding-meaning-lvalues-and-rvalues-c)

## 左值和右值的定义

左值：表示能够确认其真实地址；

右值：表示一个临时变量；或者只拥有很短的生命周期的变量；



- 等号

由于左值有对应的真实地址，因此能够对其进行幅值，而右值为一种临时变量，不能对其进行幅值；

`因此，在进行赋值操作的时候，等式的左边必须为左值，等式的右边可以为左值或者右值；换句话来说：左值既能出现在左边又能出现在右边，右值只能出现在等号的右边；`

合法的操作：

```c++
int a = 2;	// a 为左值；2 为右值
int b = a; 	// b 为左值；a 为右值
int* c = &b;// c 为左值；&b 为右值
```

不合法的操作：

```c++
int x = 1;
int 1 = x;	// 不合法！1 为右值不能放在等号左边
```

- 取地址

由于左值有对应的具体地址，右值没有；那么只能对左值进行取地址操作；

```c++
int x = 1;
int* y = &x;	// 合法的操作；x为左值，&对左值进行操作；y为左值；&x 为右值

int* z = &100;  // 不合法操作；100为右值；不能对右值进行取地址；
```

- 函数返回左值和右值

返回值为引用的时候（比如 `int&`），返回值为左值，返回值为其他情况的时候为右值；

```c++
int set_value(){
    return 1;
}

int& set_global(){
    return global;
}

int main(){
    // set_value() = 1; // error!
	set_global() = 1;   // correct!
}

```

- 左值到右值的转化

C++中的`+`操作符接受两个右值，返回一个右值；但是下面这个代码仍然是合法的；

```c++
int x = 1;
int y = 2;
int z = x + y;
```

`+`操作符接受了两个左值，表示接受了左值到右值的转换；



- 左值引用

C++中能够声明一个变量的引用；改变该引用的值就相当于改变引用所指向的变量；能够通过下面的语句声明一个引用：

```c++
int x = 1;
int& y = x;	// 声明了 x 的引用 y

int& z = 1;	// error!
```

那么如果我一步到位直接将`1`给引用`z`呢：不行，因为z需要指向一个拥有具体的地址的变量，而不是没有具体地址的右值；

这里其实也是表示，不能直接将右值转化为左值；如果允许这样做，则可以通过其引用来更改数值常量的值。没有任何意义；

```c++
void func(int& x){
    // ... do something
}

int main(){
    func(10); 	// error!
    // 下面的语句是正确的
    int x = 10;
    func(x);
}
```



- 左值常量引用

上面这段代码使用GCC进行编译，报错的结果为：

![image-20240511105729793](/assets/image-20240511105729793.png)

其报错为参数`x`为非常量的左值引用；根据C++语言规范，您可以将右值绑定到常量左值。

```c++
void func1(const int& x){
    // ... do something
}

int main(){
    func1(10);	// 正确！
}
```