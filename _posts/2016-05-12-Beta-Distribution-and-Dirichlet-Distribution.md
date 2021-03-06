---
layout: post
title:  "Beta Distribution and Dirichlet Distribution"
date: 2016-05-12 21:04:02 +0800
categories: "math"
author: xxiieao
---

最近在学习机器学习时碰到了两个以前并不太熟悉的概念——Beta分布与Dirichlet分布。当我直接看理论知识看到以下信息时——“Beta分布是伯努利分布（Bernoulli distribution）和二项式分布（Binomial Distribution）的共轭先验分布”，我发现我很难理解这句话，更难理解这话的重要意义。实际上，这两个分布非常重要，我很奇怪之前居然一直没有什么了解。然后我意识到，虽然我在之前的学习中学会了贝叶斯法则的用法，但是并没有理解贝叶斯法则的意义。这也导致了我在之前的博文中犯了一个错误——我认为贝叶斯过滤器并不属于机器学习的范畴，因为当时的我认为这个模型在运行时并没有修正任何预设参数。

#### 贝叶斯法则基础 ####

在许多概率学的课本上，贝叶斯法则的描述简单而粗暴。往往就是下面这个简单的公式

$$ P(A|B) = \dfrac {P(B|A) * P(A)}{P(B)} $$

比较经典的解释是P(A\|B)是已知B事件时，A事件的条件概率。所以这个公式描述了事件A和事件B条件概率的关系。这种理解是非常教条的，实际更好地理解贝叶斯法则，需要理解这个法则提出是为了解决什么问题。

这里我们重新来认识一下这两个变量的含义。A代表我们的认识，也可简单的理解为一个模型。P(A)代表这个模型成立的概率，或者叫做先验概率。B代表一个证据，P(B\|A)被称为似然率（PS：P(B\|A)/P(B)被称为标准似然率），可以理解为当模型A成立时，出现证据B的概率。整个公式的样子变成了下面的样子

$$ 验后概率 \propto 先验概率 × 似然率 $$

总结一下上面式子，在我们观测到证据B前，A模型成立的概率为P(A)。当我们观测到证据B后，相当于对模型A进行了验证，现在A模型的成立概率发生了变化变成了P(A\|B)。这也说明了实际上贝叶斯法则的中心思想是通过新证据来更新我们的认识。

#### 贝塔分布(Beta Distribution)

现在我们可以进入正题来讨论贝塔分布的作用了。假设我现在有一枚硬币，我想知道抛该硬币获得正面的概率是多少（需要说明的是这枚硬币也许是不均匀，不能粗暴地说50%）。在条件允许的情况下，我可以抛掷足够次数直到概率稳定收敛到一个值。但在实际应用中，这一点也很可能无法满足，你只能抛少数几次，显然这个时候要求给出概率精确的值是不可能的，但是我们仍然可以给出可靠的统计推断即正面概率在某个区间内的分布，或者说我们可以获得一个概率的概率分布。

想象以下情况，我先拿这个硬币抛掷了一次，结果是正面。如果我们机械地使用最大似然法则来估算这个硬币正面的概率，那么结果是100%。这显然是不合理的，显然这个时候比较合理的想法是，由于获得了一次正面，那么我猜这个硬币正面的概率比50%高那么一点点。于是我再抛一次，又是正面，那么我猜这个硬币正面的概率又比上次50%高一点点的那个数值再高一点点。如果我继续抛下去，获得了一连串的正面（10次以上），我猜这个硬币正面的概率显然是严重地高于50%的。

在这个过程中，贝塔分布正好可以帮助我们做到这一点。贝塔分布只有两个参数$$\alpha$$和$$\beta$$，一个是成功次数，一个是失败的次数。在没有抛硬币之前，我认为正面的先验概率应该刚好是50%，这里假设正面和失败都是50次（可以认为正面就是成功）。可以发现正面概率的分布曲线峰（下图红线），随着我们一次又一次得到正面的结果逐渐向右侧移动了（下图蓝线）。

{% highlight R linenos %}
# R代码
x = seq(0, 1, .01)
plot(x, dbeta(x, 50, 50), typ = 'l', col = 'red', ylim = c(0, 10), xlab = '正面概率', ylab = '概率密度')
points(x, dbeta(x, 50 + 10, 50), typ = 'l', col = 'blue')
legend(0.6, 10, c('正面=50，背面=50','正面=60，背面=50'), col = c('red', 'blue'), lty = 1, bty = 'n')
{% endhighlight %}

![pictrue](http://ww3.sinaimg.cn/mw690/6daafd01gw1f3swwtcu2bj20dc0dc3yz.jpg)

你可能会注意到，峰值的移动速度和我们一开始设置的$$\alpha$$和$$\beta$$值有很大的关系。这个想法很正确，关于这两个值的设置的确有一些小窍门在里面。这里我拿一个实际的例子来解说，比如知乎回答的赞同比例排序。这里我们先不考虑答案提交时间的影响，只考虑赞同的比例。假设一个答案A获得1000个赞同和200个反对，另一个答案B获得20个赞同和0个反对。哪个答案的赞同比例更高？后一个答案显然是100%，不过你也许会争辩说后一个答案取样也太少了。没错后一个答案显然没有足够的数量说明它的赞同比例。为了比较者两个答案，我们先要为他们选择同一个基准（验前概率）。显然用平均赞同和反对数量来做这个基准是非常合理的。假设知乎的平均赞同数是2000，反对数是500。那么通过贝塔分布我们可得到一个先验分布，这个就是基准（下图红线，只显示了0.75到0.85范围）。把第一个答案的1000个赞同和200个反对加上去，我们就获得了答案A赞同率的分布（下图蓝线）。同理也能获得答案B的赞同率分布（下图绿线）。

{% highlight R linenos %}
# R代码
x = seq(0, 1, .001)
plot(x, dbeta(x, 2000, 500), typ = 'l', col = 'red', ylim = c(0, 80), xlim = c(0.75, 0.85), xlab = '赞同率', ylab = '概率密度')
points(x, dbeta(x, 2000 + 1000, 500 + 200), typ = 'l', col = 'blue')
points(x, dbeta(x, 2000 + 20, 500 + 0), typ = 'l', col = 'green')
legend(0.82, 75, c('平均赞同分布','答案A','答案B'), col = c('red', 'blue','green'), lty = 1, bty = 'n')
{% endhighlight %}

![pictrue](http://ww4.sinaimg.cn/mw690/6daafd01gw1f3swte7cjlj20dc0dcdge.jpg)

现在我们就可以比较答案A和B的赞同率了，两个分布的具体的比较方法不是这篇blog的重点，这里就不再讨论了。

#### 狄利克雷分布(Dirichlet Distribution)

狄利克雷分布简单来说就是贝塔分布的多结果版本。将前面抛掷的硬币换成一个色子的话，六个面的结果就构成狄利克雷分布。

#### 关于共轭

通过前面，我们明白了贝塔分布和狄利克雷分布的应用。但是在我开篇的问题中还有一个非常重要的概念——共轭。共轭分布是指不同分布之间具有相似的函数形式。简单的来说，随着参数的调整可以使得共轭的两个分布呈现出一样的概率密度曲线。在上面的抛硬币的例子就是——描述结果的Beta分布可以转换成二项式分布。在上面的例子中，验前分布与验后分布共轭是非常重要的，这样的话每当我们获得一个新的观测数据可以马上用于更新。因为无论多少次更新，验后分布的函数形式是不变的。
