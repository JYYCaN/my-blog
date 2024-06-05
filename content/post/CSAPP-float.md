---
title: "计算机对有理数的表示——浮点数"
date: 
lastmod: 
draft: true
categories: ["CSAPP", "计算机基础", "C/C++"]
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

# 浮点数

Ref

- [L04_floating_point_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hf4y1P7qW?p=18&vd_source=4d9291fb6cdaad60d2276064508b2ad5)
- [15-213/15-513 Introduction to Computer Systems: Schedule (cmu.edu)](https://www.cs.cmu.edu/~213/schedule.html)



## 1、Fractional Binary Numbers 表示
![image-20240603205555943](/assets/image-20240603205555943.png)

公式：

$\mathrm{value} = \sum_{k=-j}^{i} \times 2^k$

- 通过移位来实现乘除
- $0.1111111..._2$​ 也小于 1.0

缺点：

- 只能表示满足下面公式的数据：$\frac{x}{2^k}$
- 小数点在什么位置需要确认，数据的范围有限制（当有非常大和非常小的数据相加时，不不能进行表示）



## 2、浮点数表示

![image-20240604093301360](/assets/image-20240604093301360.png)



- 使用科学计数法对数据进行表示
- S：Sign，符号位
- Exp：指数位
- Frac：小数位

float32:

- S: 1 bit
- Exp：8 bit
- Frac：23 bit



double

- S：1 bit
- Exp：11 bit
- Frac：52 bit



$\mathrm{value} = (-1)^S \times M \times 2^E$​​



浮点数能表示的所有数据分为 Normalized Value 和 Denormalized Value

>
> Normalized Value
>
> - 当 ${Exp} \neq 000...0$ 并且 ${Exp} \neq 111...1$
> - Normalized Value 中： $E = Exp - Bias$​
> - 其中 $Bias = 2^{k-1} - 1$，$k$ 为 $Exp$ 的位数
>   - float 的 Bias 为 127（Exp：1...254，E：-126...127）
>   - double 的 Bias 为 1023（Exp：1...2046, E：-1022...1023）
> - 小数部分表示为$M = 1.xxx...x_2$
> - 其中 $.xxx...x_2$​ 通过 Frac 来进行表示
> - 小数部分最小值为：000...0（M = 1.0）
> - 小数部分最大值为：111...1（M = 2.0 - $\epsilon$​）

例子：

float F = 15213.0;

- $15213_{10} = 11101101101101_2 = 1.1101101101101_2 \times 2^{13}$

- $Frac = 1101101101101$
- $M = 1.1101101101101$

- $E = 13_{10}$
- $Bias = 127_{10}$
- $Exp = 140_{10} = 10001100_{2}$

因此结果为：

0 (s) 1000 1100 (exp) 1101 1011 0110 1000 0000 000 (frac)



> Denormalized Value
>
> - 当 $Exp = 000.0$ 的时候
> - Denormalized Value：$E = 1 - Bais$​
> - $M = 0.xxx...x_2$，其中 $xxx...x_2$ 通过 Frac 表示
> - $Exp = 000...0$，$Frac=000.0$ 表示 0
> - 因此浮点数拥有 $+0$ 和 $-0$
> - 当 $Exp = 000...0$，$Frac \neq 000...0$​​ 的时候，浮点数为非常接近 0 的数；
> - 并且这些数据是均匀分布的



>
> +Inf 和 -Inf
>
> - 当 $Exp = 111...1$，$Frac = 000...0$ 时表示无穷
> - 表示超出了浮点数的表示范围（例如：$1.0/0.0 = +Inf$​）
>
> NaN
>
> - 当$Exp = 111..1$，$Frac \neq 000.0$​时表示NaN（Not a Number）
> - 例如：$sqrt(-1)$​



基于此，浮点数能够表示的数据能够通过数轴表述如下：

![image-20240604100758691](/assets/image-20240604100758691.png)

## 3、Example 8比特浮点数：

Sign：1位、Frac：3位、Exp：4位

| s    | exp  | frac |  E   |                            value                             |  type  |            |
| ---- | :--: | :--: | :--: | :----------------------------------------------------------: | :----: | ---------- |
| 0    | 0000 | 000  |  -6  |               $2^{-6}\times 2^{-3} \times 0=0$               | Denorm | 最小正数   |
| 0    | 0000 | 001  |  -6  |        $2^{-6}\times 2^{-3} \times 1 = \frac{1}{512}$        | Denorm |            |
| 0    | 0000 | 010  |  -6  |       $2^{-6} \times 2^{-3} \times 2 = \frac{2}{512}$        | Denorm |            |
| 0    | 0000 | 011  |  -6  |         $2^{-6} \times 2^{-3}\times 3=\frac{3}{512}$         | Denorm |            |
| ...  | ...  | ...  | ...  |                             ...                              |  ...   |            |
| 0    | 0000 | 111  |  -6  |       $2^{-6} \times 2^{-3} \times 7 = \frac{7}{512}$        | Denorm | 最大Denorm |
| 0    | 0001 | 000  |  -6  | $2^{-6} \times (1 + 2^{-3} \times 0) = \frac{1}{64} = \frac{8}{512}$ |  Norm  | 最小 norm  |
| 0    | 0001 | 001  |  -6  |    $2^{-6} \times (1 + 2^{-3} \times 1) = \frac{9}{512}$     |  Norm  |            |
| 0    | 0001 | 010  |  -6  |    $2^{-6} \times (1 + 2^{-3} \times 2) = \frac{10}{512}$    |  Norm  |            |
| 0    | 0001 | 011  |  -6  |    $2^{-6} \times (1 + 2^{-3} \times 3) = \frac{11}{512}$    |  Norm  |            |
| 0    | 0001 | 100  |  -6  |    $2^{-6} \times (1 + 2^{-3} \times 4) = \frac{12}{512}$    |  Norm  |            |
| ...  | ...  | ...  | ...  |                             ...                              |  ...   |            |
| 0    | 0001 | 111  |  -6  |      $2^{-6}\times (1+2^{-3}\times7) = \frac{15}{512}$       |  Norm  |            |
| 0    | 0002 | 000  |  -5  |      $2^{-5}\times(1+2^{-3} \times 0) = \frac{16}{512}$      |  Norm  |            |
| 0    | 0002 | 001  |  -5  |      $2^{-5}\times(1+2^{-3} \times 1) = \frac{18}{512}$      |  Norm  |            |
| 0    | 0002 | 010  |  -5  |    $2^{-5} \times (1 + 2^{-3} \times 2) = \frac{20}{512}$    |  Norm  |            |
| 0    | 0002 | 011  |  -5  |    $2^{-5} \times (1 + 2^{-3} \times 3) = \frac{22}{512}$    |  Norm  |            |
| 0    | 0002 | 100  |  -5  |    $2^{-5} \times (1 + 2^{-3} \times 4) = \frac{24}{512}$    |  Norm  |            |
| ...  | ...  | ...  | ...  |                             ...                              |  ...   |            |
| 0    | 0002 | 111  |  -5  |     $2^{-5}\times (1 + 2^{-3}\times 7) = \frac{30}{512}$     |  Norm  |            |
| 0    | 0003 | 000  |  -4  |    $2^{-4} \times (1 + 2^{-3} \times 0) = \frac{32}{512}$    |  Norm  |            |
| 0    | 0003 | 001  |  -4  |    $2^{-4} \times (1 + 2^{-3} \times 1) = \frac{36}{512}$    |  Norm  |            |
| 0    | 0003 | 010  |  -4  |    $2^{-4} \times (1 + 2^{-3} \times 2) = \frac{40}{512}$    |  Norm  |            |
| ...  | ...  | ...  | ...  |                             ...                              |  ...   |            |
| 0    | 0003 | 111  |  -4  |    $2^{-4} \times (1 + 2^{-3} \times 7) = \frac{60}{512}$    |  Norm  |            |
| 0    | 0004 | 000  |  -3  |   $2^{-3} \times ({1 + 2^{-3} \times 0}) = \frac{64}{512}$   |  Norm  |            |
| 0    | 0004 | 001  |  -3  |   $2^{-3} \times (1 + 2^{-3} \times  1) = \frac{72}{512}$    |  Norm  |            |
| ...  | ...  | ...  | ...  |                             ...                              |  ...   |            |
| 0    | 1110 | 000  |  7   |           $2^7 \times (1 + 2^{-3} \times 0) = 128$           |  Norm  |            |
| 0    | 1110 | 001  |  7   |           $2^7 \times (1 + 2^{-3} \times 1) = 144$           |  Norm  |            |
| 0    | 1110 | 010  |  7   |           $2^7 \times (1 + 2^{-3} \times 2) = 160$           |  Norm  |            |
| 0    | 1110 | 011  |  7   |           $2^7 \times (1 + 2^{-3} \times 3) = 176$           |  Norm  |            |
| 0    | 1110 | 100  |  7   |           $2^7 \times (1 + 2^{-3} \times 4) = 192$           |  Norm  |            |
| 0    | 1110 | 101  |  7   |           $2^7 \times (1 + 2^{-3} \times 5) = 208$           |  Norm  |            |
| 0    | 1110 | 110  |  7   |           $2^7 \times (1 + 2^{-3} \times 6) = 224$           |  Norm  |            |
| 0    | 1110 | 111  |  7   |           $2^7 \times (1 + 2^{-3} \times 7) = 240$           |  Norm  | 最大 norm  |
| 0    | 1111 | 000  |  -   |                             +inf                             |        |            |
| 0    | 1111 | ~000 |  -   |                             NAN                              |        |            |

下面是对 8bit float 数据进行的可视化，可以看出：

==浮点数的数据在0附近的表示精度越低，越大和越小的数据表示的精度越低==

![image-20240531101858778](/assets/image-20240531101858778.png)



## 4、浮点操作

浮点加法：$x + y = Round(x + y)$

浮点乘法：$x \times y = Round(x \times y)$

- 浮点操作可能越界（Overflow）;
- 可能需要舍入（Round）；

舍入方法：

- 向零取整（Towards zero）：舍去小数部分
- 向下取整（Round down）：向正无穷取整
- 向上取整（Round up）：向负无穷取整
- 向偶数取整（default）：哪边近向哪边取整，当而后两边同样近时向偶数那边取整（现有浮点数的基于该思想实现）（因为要保持统计上的均值）

|            | 1.40 | 1.60 | 1.50 | 2.50 | -1.50 |
| ---------- | ---- | ---- | ---- | ---- | ----- |
| 向零取整   | 1    | 1    | 1    | 2    | -1    |
| 向下取整   | 1    | 1    | 1    | 2    | -2    |
| 向上取整   | 2    | 2    | 2    | 3    | -1    |
| 向偶数取整 | 1    | 2    | 2    | 2    | -2    |



### 浮点乘法（FP Multiplication）

$(-1)^{s1} M1 \ 2^{E1} \times (-1)^{s2} M2 \ 2^{E2}$

- Sign $s = s1$ ^ $s2$
- $M = M1 \times M2$
- $E = E1 + E2$



还需要对结果进行修正：

- 当 $M \ge 2$ 时，将 M 右移一位，并且 E 加 1
- 当 E 溢出的时候，越界



### 浮点数操作需要注意的事项

- 浮点加法具有交换律：$x + y = y + x$
- 浮点加法不具有结合律：$(x + y) + z \neq x + (y + z)$
  - 例如：$(3.14 + 1e^{10}) - 1e^{10} = 0；3.14 + (1e^{10} - 1e^{10}) = 3.14$
  - 大数据和小数据相加的时候容易造成较大误差
- 浮点乘法具有交换律：$x \times y = y \times x$​
- 浮点乘法不具有结合率：$(x \times y) \times z \neq x \times (y \times z)$​
  - 例如：$(3.14 \times 1e^{20}) \times 1e^{-20} = 0；3.14 \times (1e^{20} \times 1e^{-20}) = 3.14$​



## 5、C 语言中的浮点数

C语言中提供了两种不同的浮点数

- float
- double



float/double --> int

- 截断小数部分
- 当 float/double 越界或者为 NaN 的时候设置为 TMin



int --> double

- double 的小数部分有 52 位，能够直接将 int 转化为 double 没有精度损失



int --> float

- float 的小数部分仅有 23 位，将 int 转化为 float 需要进行舍入，有精度损失



float/double --> int

- float/double 的表示范围大于 int，当这种情况发生时，将截断为 int 的最小值

- 当 float/double 转化为 int 

<!-- KaTeX -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.10.0-rc.1/katex.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.10.0-rc.1/katex.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.10.0-rc.1/contrib/auto-render.min.js"></script>

<script>
    document.addEventListener("DOMContentLoaded", function() {
      renderMathInElement(document.body, {
              delimiters: [
                  {left: "$$", right: "$$", display: true},
                  {left: "$", right: "$", display: false}
              ]
          });
    });
</script>