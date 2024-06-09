+++
title = "计算正弦函数的值域"
date = 2023-12-10T11:47:15Z
tags = ["数学", "计算机科学"]
katex = true
+++

若想要求出正弦函数（有定义域）的值域，对我们人类来说只需要用“瞪眼法”看着函数图像就能轻松解决。那计算机是如何解决这个问题的呢？以下是我提供的一个思路。

<!--more-->

首先，这里不建议重复造轮子，因为已经有现成的东西可以帮我们解决问题：
```python
>>> import mpmath as mp
>>> pi = mp.pi
>>> mp.iv.sin(mp.iv.mpf([-pi/6, 5*pi/6]))
mpi('-0.5', '1.0')
```

不过，对于那些不太想用 `mpmath` 库，而想自己实现的人，算法的实现也非常简单。请往下看吧！

为表述方便，函数 $f(x)=\sin x$ 的定义域总是 $[a,b]$。

## 数学理论
先从最简单的情况开始：如果 $b-a\ge2\pi$，那么值域总是 $[-1,1]$。

其它情况就需要知道函数局部的单调性了。这时候我们就需要正弦函数的导函数：余弦函数。

一种比较特殊的情况是：当 $\cos a\cos b\ge0$ 且 $b-a\ge\pi$ 时，函数的值域仍然为 $[-1,1]$。

当 $\cos a\cos b\ge0$ 且 $b-a\le\pi$ 时，直接代端点即可。

> 虽然以上两种情况都是 $f(x)$ 在 $x=a$ 以及 $x=b$ 的邻域上的单调性相同。前者是因为 $b-a$ 大于半个周期，期间正好有 2 个函数的极值点；后者因为 $[a,b]$ 恰好是一个单调区间的子集，所以计算值域可以直接使用端点值。

最后 2 种情况是我们通常会遇到的：

1. 当 $\cos a>0>\cos b$，$f(x)$ 有最大值 $1$
2. 当 $\cos a<0<\cos b$，$f(x)$ 有最小值 $-1$

当有最大值 $1$ 时，最小值应该是 $\sin a$ 和 $\sin b$ 中较小的那个。有最小值 $-1$ 的情况与之类似。

## 具体实现
以下 Python 代码是求正弦函数值域的具体实现：
```python
from math import sin, cos, pi

def iv_sin(a: float, b: float) -> list[float, float]:
    assert b >= a, "b must bigger than a"
    if b - a >= 2 * pi:
        return [-1, 1]
    elif cos(a) * cos(b) >= 0:
        if b - a >= pi:
            return [-1, 1]
        else:
            return sorted([sin(a), sin(b)])
    elif cos(a) > 0 > cos(b):
        max_value = 1
        min_value = min(sin(a), sin(b))
        return [min_value, max_value]
    elif cos(a) < 0 < cos(b):
        max_value = max(sin(a), sin(b))
        min_value = -1
        return [min_value, max_value]
```
