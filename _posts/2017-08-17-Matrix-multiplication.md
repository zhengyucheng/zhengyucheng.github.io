---
layout:     post
title:      "矩阵乘法"
subtitle:   " \"矩阵乘法\""
date:       2017-08-17 14:36:00
author:     "Monad"
header-img: "img/Matrix-multiplication.jpg"
tags:
    - 信息学
---

## 引入
众所周知，斐波拉契数列可以用这个公式来求：
``` C++
f[i] = f[i-1] + f[i-2]
```
这个公式可以用`O(n)`的时间复杂度求出来第`i`个斐波那契数。但是如果让你求`f[10⁹] % 10⁴`，用上述的方法是肯定会超时的。所以我们要找一种更加科学的方法来求这个斐波拉契数列的第i项。

## 矩阵
矩阵是一个什么东西呢？矩阵是一个有n行m列元素排成的举行阵列，简单来说，矩阵就是一个二维数组。我们也可以用A, B, C等大写字母来命名一个矩阵。![一个矩阵](http://latex.codecogs.com/svg.latex?A%20%3D%20%5Cbegin%7Bbmatrix%7D%209%20%26%203%20%26%2015%5C%5C%201%20%26%2011%20%26%207%20%5C%5C%203%20%26%209%20%26%202%20%5C%5C%206%20%26%200%20%26%207%20%5Cend%7Bbmatrix%7D)矩阵和整数一样，也支持加减乘操作。这里我们就先介绍一下乘法，加减法后面在介绍。

### 矩阵乘法
矩阵乘法是一种很特殊的东西，它仅仅在A的列数等于B的行数时成立。目标矩阵的大小则是A的行数×B的列数，对于目标矩阵的每一个元素C<sub>ij</sub>：![](http://latex.codecogs.com/svg.latex?C_%7Bi%2Cj%7D%3DA_%7Bi%2C1%7DB_%7B1%2Cj%7D+A_%7Bi%2C2%7DB_%7B2%2Cj%7D+...+A_%7Bi%2Cn%7DB_%7Bn%2Cj%7D%3D%5Csum_%7Br%3D1%7D%5E%7Bn%7DA_%7Bi%2Cr%7DB_%7Br%2Cj%7D)如果看不懂公式，也可以看下面生动形象的图片解释：
![矩阵乘法](http://oiq7rdgur.bkt.clouddn.com/jianshu/1/Matrix_multiplication.svg)

## 线性常系数递推
```C++
f[i] = f[i-1] + f[i-2]
```
线性常系数递推，顾名思义，“线性”，就是只有一次项，没有平方、开方、立方等操作；“常系数”，就是未知数（指`f[i-1]`和`f[i-2]`）的系数是一个常数，不是变量。
那么这个神奇的矩阵乘法真的可以帮我们解决这个问题吗？那么又怎么生成出需要的矩阵呢？![](http://oiq7rdgur.bkt.clouddn.com/jianshu/1/1.svg)如图所示，这是一个斐波那契数列。我们可以选取数列中的`[1,2]`来生成一个一行二列的矩阵。然后我们要通过何种操作才能使`[1,2]`这个矩阵生成出`[2,3]`这个新矩阵呢？![](http://latex.codecogs.com/svg.latex?%5Cbegin%7Bbmatrix%7D%201%20%26%202%20%5Cend%7Bbmatrix%7D%20%5Ctimes%20%5Cbegin%7Bbmatrix%7D%20%3F%20%26%20%3F%20%5C%5C%20%3F%20%26%20%3F%20%5Cend%7Bbmatrix%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%202%20%26%203%20%5Cend%7Bbmatrix%7D)我们可以从我们的递推公式入手，我们发现，我们要使第一个数列的2和第二个数列的2保持相等，然后使第二个矩阵的的第二个数等于第一个矩阵的数的和。稍微地想一下、推一下，就可以知道问号处应该填：![](http://latex.codecogs.com/svg.latex?%5Cbegin%7Bbmatrix%7D%200%20%26%201%20%5C%5C%201%20%26%201%20%5Cend%7Bbmatrix%7D)
然后我们在看一下如何从`[2,3]`推到`[3,5]`呢？重复上面的步骤，我们可以发现，从`[2,3]`推到`[3,5]`所需要的矩阵也是上面那个：![](http://latex.codecogs.com/svg.latex?%5Cbegin%7Bbmatrix%7D%200%20%26%201%20%5C%5C%201%20%26%201%20%5Cend%7Bbmatrix%7D)这样，我们就可以求这个特殊矩阵的`n`次幂，然后再与`[1,1]`相乘，就可以得出结果了。

## 快速幂
现在我们再来介绍一个新的东西，就是快速幂。如果我们计算n<sup>m</sup>时，如果`m`足够大，那么用`O(m)`的方法肯定会超时。因为整数满足乘法结合律，所以如图所示，![快速幂](http://latex.codecogs.com/svg.latex?%5Cunderbrace%7Ba%5Ctimes%20a%5Ctimes%20a%7D%5Ctimes%20%5Cunderbrace%7Ba%5Ctimes%20a%5Ctimes%20a%7D%5C%5C%20%5Ctext%7B%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%21%203%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%3B%20%5C%21%203%20%7D)我们可以对上面的`a⁶`拆分成`(a³)²`，然后在把`a³`拆分成`(a)²a`。这样来算幂，我们就只需要`O(log m)`的时间复杂度。

## 矩阵乘法快速幂
那么知道了上面那个规律之后，又有什么用呢，相乘起来的时间复杂度还是`O(n)`（甚至还多了矩阵乘法那`O(x³)`的时间），仍然会超时。
但是，矩阵乘法满足一个性质，就是乘法结合律，也就是：
```
(A×B)×C = A×(B×C)
```
有了这个性质之后，我们就可以对这个矩阵进行快速幂操作，这样那硕大的`O(n)`的时间复杂度就降低到`O(log n)`，加上矩阵乘法的时间，也只不过是`O(x³log n)`而已，完全可以在`1s`内得到正确答案。

## 链接
题目：[POJ 3070](http://poj.org/problem?id=3070)
我的[代码](https://github.com/YanWQ-monad/monad/blob/master/Cpp/Exam-answer/poj.org/3070.cpp)（非常玄学）
