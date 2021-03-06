---
layout: post
title:  "Generate a Random Point within a Circle"
date: 2016-04-16 01:04:36 +0800
categories: "math"
author: xxiieao
---

### 如何从圆中随机选一点

在一个给定的矩形中随机选一点，是一个非常简单的事情。先在一条边上随机选一点作其邻边的平行线，再在邻边上选一点同样作平行线，两平行线的交点便是这个随机点。而在一个圆中随机生成一点，是一个既简单又有点复杂的问题。熟悉极坐标的人，可能很快就能提出一个方案——将圆的笛卡尔坐标系转换成极坐标。先在$$[0,2\pi)$$间生成一个随机数，再在$$[0,r]$$间随机生成一个数字，这两个数值将确定圆上一个唯一的点。

到目前为止，随机选一点的目的似乎已经达到，至少从数学上来说是的。不过鉴于日常用语的模糊性，对随机这个词的不同理解会对算法的设计产生巨大的不同。比如，如果在某个随机抽奖活动中号码位数为偶数的用户比奇数用户抽到的概率高2倍的话，用户估计要骂娘了。随机只是指一种不确定性而已，而日常生活用语中的随机二字常常包含一个隐含的前提——“等概率”，就是我们常常说的均匀。

那么上面这个算法均匀吗？具体一点，在下图中，点落在A阴影内和B阴影内的概率相同吗？显然如果A和B对应的圆心角相等，对应的半径截长也相等，那么落到A和B的概率显然是相等的。上面算法已经是随机的了，但是还不够均匀。

![pictrue](http://ww2.sinaimg.cn/mw690/6daafd01gw1f2yoolx14rj208t0b8a9z.jpg)

要达到均匀的效果，就需要达到以下条件——随机点落在圆中任意两个面积相等图形的概率是相等的。网上有很多关于在圆上均匀取一点的算法回答——先在$$[0,2\pi)$$间生成一个均匀随机数，再在$$[0,1]$$间均匀随机生成一个数字，然后对这个数字取开方然后乘以半径，再由这两个数值确定圆上一个唯一的点。不过大多数回答并没有论述为何做是有效的，下文将尝试从证明的角度来说明这个算法的有效性（注意：下文并不是严格的证明）。

要做到任意图形这一点，我们可以先对圆进行微分，将圆先分成类似B那样的细小微粒，这些小颗粒的面积都相等，可以认为面积相等的任意图形都是由数量相等的小颗粒组成。那么我们只要保证随机点落在每一个小颗粒的概率相等就可以保证落在面积相等任意图形里的概率也是相等的了。

为了方便计算，我们假设这些微粒对应的圆心角都是相等，弧度均为$$\phi$$，离圆心的距离为d，自身在半径上的投影长度为i。

$$S = \dfrac {((d + i)^2 - d^2) * \pi * \phi}{2 * \pi} = \dfrac{(i^2 + 2*d*i) * \phi}{2} $$

显然这个i是不固定的，为了让微粒的面积相等，离圆心越近i就会越大。容易知道贴近圆心的那个微粒$$P_0$$的面积是$$i_0^2 * \phi / 2$$。我们可以这个微粒的面积作为标准来计算，任意微粒$$P_d$$的$$i_d$$与d的关系。

$$ i_d^2 + 2 * d * i_d = i_0^2 $$

由于d>0，可知以上二元一次方程的正根只有一个$$ i_d = \sqrt{d^2 + i_0^2} - d $$

到这一步我们解决了微粒的面积相等问题，下一步需要解决概率相同问题。由于圆心角相等，落在$$P_0$$和其它$$P_d$$内的概率比例，与其半径投影长度成正比。也就说，需要点落在$$[0, i_0]$$的概率与落在$$[d, \sqrt{d^2 + i_0^2}]$$的概率相等。然后我们可以看一看上面算法是不是等概率地落在这两个区间呢？

当随机数落在在$$[0, i_0^2]$$之间时，最终对应的点会落在$$[0, i_0]$$范围内，而随机数落在$$[d^2, d^2+i_0^2]$$之间时，最终的点会落在$$[d, \sqrt{d^2 + i_0^2}]$$。显然区间$$[0, i_0^2]$$与区间$$[d^2, d^2+i_0^2]$$的长度相等，概率是相等的。
