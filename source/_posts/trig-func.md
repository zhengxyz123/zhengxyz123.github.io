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

# $A\sin(\omega x+\varphi)+C$性质
我们可以总结一些三角函数的性质，如下表：

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
以函数$f(x)=A\sin(\omega x+\varphi)+C$为例，当定义域为$\mathbb{R}$时，它的值域就是$[A+C, A-C]$；若定义域不为$\mathbb{R}$，则通过多次换元来计算值域：

1. $t=\omega x+\varphi$
2. $h=\sin t$（可通过绘制函数图像确定范围，并检查开闭）
3. $f(x)=Ah+C$
