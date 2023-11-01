---
title: 三角函数笔记
date: 2023-09-07 21:30:24
tags:
    - 数学
    - 高中数学
mathjax: true
---

高三开学已经一周了，数学老师也开始了他的总复习计划。他首先给我们复习的是三角函数部分。在此，我回忆了三角函数的各种知识，并结合复习时的内容，留下了这篇笔记。

<!-- more -->

以下是我使用的三角函数章节的思维导图，它简明扼要的描述了各个知识点之间的关系，前三级子分支有各自对应的标题。

![思维导图](mindmap.png)

# 化简方向
“化简方向”这一分支主要讲如何求含三角函数的值域。
> 虽然有那么一部分是真的在教你怎么化简一个三角函数，但是其它部分就只告诉你应该怎样求值域。这个名字实在是太不合适了。

## 整式
阅读思维导图，“整式”分支可利用辅助角公式，使函数仅含$\sin$；或是换元成二次函数。

那对于下面的函数，我们应该走哪条路呢？

- $f_1(x)=\sin2x-2\cos^2x$
- $f_2(x)=\sin x\cos(x-\dfrac{\pi}{3})$
- $f_3(x)=\cos4x-2\sin^2x$

毋庸置疑，应该把式子全部展开后再判断，那么$f_2(x)=\dfrac{1}{2}\sin x\cos x+\dfrac{\sqrt{3}}{2}\sin^2x$。

观察三角恒等变换的结果，我们会发现${\color{red}\text{三角函数的次数}}\times{\color{blue}\text{角的系数}}$是不变量。

\\[\sin^{\color{red}2}{\color{blue}2}x=1-\cos^{\color{red}2}{\color{blue}2}x=\dfrac{1-\cos{\color{blue}4}x}{2}\\]

那么我们就可以用该不变量（我们就称之为“角次”吧）来判断应该使用何种方法。

### 辅助角公式
当角次相同时，我们可以使用辅助角公式化简。$f_1(x)$和$f_2(x)$适用这个方法。
> 以$f_1(x)$为例，角次是一项一项计算的：$\sin2x$的角次是$2$；$2\cos^2x$的角次也是$2$。

若想要使用辅助角公式化简，那么三角函数的次数应该全部化为$1$，再运用辅助角公式：

- $f_1(x)=\sin2x-\cos2x-1=\sqrt{2}\sin(2x-\dfrac{\pi}{4})-1$
- $f_2(x)=\dfrac{1}{4}\sin2x-\dfrac{\sqrt{3}}{4}\cos2x+\dfrac{\sqrt{3}}{4}=\dfrac{1}{2}\sin(2x-\dfrac{\pi}{3})+\dfrac{\sqrt{3}}{4}$

如果可以使用计算器，则可以讨巧地求得辅助角公式的结果：在“虚数”模式下将一个虚数转换为其三角形式可以看作使用了辅助角公式。

### 二次
当角次不相同时，我们就应该换元。

## 分式
以下介绍如何求带有分数线的三角函数的值域。

### $\frac{I}{I}$型
对于分子和分母一个含$\sin$，另一个含$\cos$，我们可以把值域问题转换为方程有解。
> 方程有解和求值域在本质上是一样的，大家可以用自己的小脑筋好好思考一下。

同时，在解析几何中，我们也可以将值域问题转换为求斜率的范围。

### 万能公式
使用万能公式（也叫正切半角公式），可以将函数名称统一成$\tan$以便换元，这是最后的手段，同时也是通用的手段。

# $A\sin(\omega x+\varphi)+C$性质
我们可以总结一些三角函数的性质，正弦函数的最常用，如下表：

| 性质 | $f(t)=\sin t$ | $f(t)=\cos t$ | $f(t)=\tan t$ | $f(t)=\cot t$ |
|:----:|:-------------:|:-------------:|:-------------:| :-----------: |
| 定义域 | $\mathbb{R}$ | $\mathbb{R}$ | $t\neq k\pi+\frac{\pi}{2}$ | $t\neq k\pi$ |
| 值域 | $[-1,1]$ | $[-1,1]$ | $\mathbb{R}$ | $\mathbb{R}$ |
| 最大值时的$t$ | $2k\pi+\dfrac{\pi}{2}$ | $2k\pi$ | 无 | 无 |
| 最小值时的$t$ | $2k\pi-\dfrac{\pi}{2}$ | $2k\pi+\pi$ | 无 | 无 |
| 奇偶性 | 奇函数 | 偶函数 | 奇函数 | 奇函数 |
| 单调递增区间 | $[2k\pi-\dfrac{\pi}{2},2k\pi+\dfrac{\pi}{2}]$ | $[2k\pi-\pi,2k\pi]$ | $(k\pi-\dfrac{\pi}{2},k\pi+\dfrac{\pi}{2})$ | 无 |
| 单调递减区间 | $[2k\pi+\dfrac{\pi}{2},2k\pi+\dfrac{3\pi}{2}]$ | $[2k\pi,2k\pi+\pi]$ | 无 | $(k\pi,k\pi+\pi)$ |
| 周期 | $\dfrac{2\pi}{\|\omega\|}$ | $\dfrac{2\pi}{\|\omega\|}$ | $\dfrac{\pi}{\|\omega\|}$ | $\dfrac{\pi}{\|\omega\|}$ |
| 对称轴 | 直线$t=k\pi+\dfrac{\pi}{2}$ | 直线$t=k\pi$ | 无 | 无 |
| 对称中心 | $(k\pi,0)$ | $(k\pi+\dfrac{\pi}{2},0)$ | $(\dfrac{k\pi}{2},0)$ | $(\dfrac{k\pi}{2},0)$ |
| 零点 | $t=k\pi$ | $t=k\pi+\dfrac{\pi}{2}$ | $t=k\pi$ | $t=k\pi+\pi$ |

在使用该表时，令括号内的$\omega x+\varphi$为$t$，通过换元的方法求得函数的单调区间等性质。

## 值域
以函数$f(x)=A\sin(\omega x+\varphi)+C$为例，当定义域为$\mathbb{R}$时，它的值域就是$[-A+C, A+C]$；若定义域不为$\mathbb{R}$，则通过多次换元来计算值域：

1. $t=\omega x+\varphi$
2. $h=\sin t$（可通过绘制函数图像确定范围，并检查开闭）
3. $f(x)=Ah+C$