---
layout: post
title:  "How to Draw Fractal"
date: 2015-04-05 21:10:02
categories: "math"
author: xxiieao
---

先介绍一下分形吧。百度上面关于分形的定义比较罗嗦，其实分形最重要的特征就是自相似，说白了就是局部放大之后会和整体非常相似。以前在网上看到过许多分形的图片，觉得非常漂亮，但是也一直没有动手画过。昨天看了一些关于分形生长的论文，里面提到反复迭代函数$$f(z) = z^2 + c$$就可以画出各种各样的分形。这个方程如此简单，让我觉得可以很轻松地在R中实现。不过很快我就犯了错误，看到论文的论述我的第一反应是$$f(z)$$的轨迹图是个分形。不过R上实践的结果根本不是分形。于是到matrix67的博客上看到了关于如何使用$$f(z) = z^2 + c$$画分形的解释。

首先需要明确的第一点变量$$z$$是复数，当然$$c$$也是复数。

其次构成分形的不是函数$$f(z) = z^2 + c$$的轨迹，而是$$f(z)$$的发散速度。$$z$$是不同数值时，即初始自变量不同$$f(z)$$发散的速度是不同的。而在计算模拟中，为了方便处理，一般定义为超过某个阈值需要的迭代计算次数。例如当$$z = 0.6 + 0.5i$$，$$c = 1 + 0.5i$$时，阈值为2时，需要迭代两次，那么这个2就作为$$x = 0.6 $$和$$ y =0.5$$这个点发散速度。对某个区域的点均进行计算再根据迭代次数进行计算就可以得到一张分形图。下面是R的代码，有兴趣的同学可以试一下

下面是当$$c = 0.45 - 0.1428i$$时画出来的图片

![picture](http://ww2.sinaimg.cn/mw690/6daafd01gw1exia8nn971j20ad09c74e.jpg)
![picture](http://ww1.sinaimg.cn/mw690/6daafd01gw1exia8o83rnj20ad09c3ym.jpg)
![picture](http://ww4.sinaimg.cn/mw690/6daafd01gw1exia8osu7cj20ad09caa5.jpg)

下面附上R的代码

{% highlight R linenos %}
complex_matrix_int = function(interval, x_start, x_end, y_start, y_end){
  #输入间隔，x轴和y轴的起始坐标
  #输出一个由x坐标和y坐标构成的复数矩阵
  x_distance = x_end - x_start
  y_distance = y_end - y_start
  xaxis = seq(x_start, x_end, interval)
  yaxis = seq(y_start, y_end, interval)
  x_rep_times = length(yaxis)
  y_rep_times = length(xaxis)
  matrix(complex(real = rep(xaxis, x_rep_times),
                 imaginary = rep(yaxis, each = y_rep_times)),
                 x_rep_times, y_rep_times, byrow = T)
}

iterator = function(complex_matrix, constant){
  #输入一个由x坐标和y坐标构成的复数矩阵和常数c
  #输出一个与输入对应的矩阵，每个元素代表对应坐标超过2时的迭代次数
  res = matrix(0, nrow(complex_matrix), ncol(complex_matrix))
  for(i in 1:100){ #因为有的点是收敛而非发散永远不会超过2因此设置迭代上限为100次
    res = res + as.numeric(abs(complex_matrix) <= 2)
    complex_matrix = complex_matrix^2 + constant
  }
  res
}

fractal_plot = function(interval = 0.005,
                        x_start = -1.5, x_end = 1.5,
                        y_start = -1.5, y_end = 1.5,
                        constant = 0.45-0.1428i){
  #调用complex_matrix_int和iterator
  complex_matrix = complex_matrix_int(interval, x_start, x_end, y_start, y_end)
  iter_times_matrix = iterator(complex_matrix, constant)
  #并对迭代次数的矩阵进行染色
  filled.contour(x = seq(x_start, x_end, interval),
                 y = seq(y_start, y_end, interval),
                 z = t(iter_times_matrix),
                 col = rainbow(121),
                 levels = pretty(c(0, 100), 101))
}

fractal_plot(0.005, -1.5, 1.5, -1.5, 1.5, 0.45-0.1428i) #第一张图
fractal_plot(0.001, 0.1, 0.6, -0.6, -0.1, 0.45-0.1428i) #第二张图
fractal_plot(0.00025, 0.3, 0.4, -0.5, -0.4, 0.45-0.1428i) #第三张图
{% endhighlight %}