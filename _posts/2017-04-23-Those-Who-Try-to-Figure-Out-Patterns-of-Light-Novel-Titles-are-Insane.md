---
layout: post
title:  "Those Who Try to Figure Out Patterns of Light Novel Titles are Insane"
date: 2017-04-23 10:04:31 +0800
categories: "data analysis"
author: xxiieao
---

《那些在轻小说标题里找规律的人脑子肯定有问题》

本文其实源于一段简短的网上聊天。上个星期我和网友聊天，对方表示现在的轻小说越来越无聊了，而我作为一个轻小说阅读甚少的人，为了不冷场决定卖弄一下不知道从哪里听来的二手知识。于是乎我表示——对啊对啊，不是说标题越长小说越烂么。可我万万没想到，对方对这个意见表示了反对，并且举出了一些例子表示这些小说标题虽然很长但是很好看。那么问题就来——轻小说真的标题越长越烂吗？为了回答这个问题，首先需要明确问题本身，使得这个问题能够被证实或者被证伪。首先是作品评价的量化，这个还比较好解决，一些书评网站上就有现成的小说打分系统。其次因为能很轻易地举出很多优秀的长标题轻小说，也能举出很多糟糕的长标题轻小说，显然需要回答的不是某三两部作品的比较，而是轻小说这个整体的统计。那么这个问题可以以另一种更加明确的方式提出——轻小说的标题长度与小说评分之间存在显著的负相关吗？

明确了问题，我们便可以着手进行数据的收集。书评网站有很多（如豆瓣），不过基于我们的数据要求，这个网站的数据最好是和轻小说相关性高的，否则还要把轻小说挑出来。另外再加上一点，我希望了解的是中国读者对轻小说的感想，因此日本的网站也不在我的考虑之列。最后我选定了Bangumi，这是一个ACGN网站，里面的小说基本上全是轻小说非常符合要求，而且书籍搜索列表页面就有书籍的评分，因此都不需要爬去书籍的详细，又进一步减少了工作量。最后，我选择爬取了记录在Bangumi上2007-2016年间出版的小说标题、出版时间、用户评论数量和用户评分这四种数据作为分析材料。爬取到的数据首先以json格式存在本地文件中，再通过Python清洗后载入Mysql中。

数据就绪，可以开始着手进行统计了，不过需要先解决一个相当麻烦的问题——标题长度的定义。读过轻小说的人应该知道，轻小说的标题那叫一个五花八门——英语、外来语、片假名、平假名甚至是标点符号齐飞，相比之下副标题到底算不算进标题长度已经不是个事了。可能有些人还没完全听懂我在说什么，那么我慢慢举例：先来一个简单的标题——《空の境界 未来福音》，这个我想大部分人都会同意按照字符数认为标题长度是8（不计空格）或者9；然后来一个稍稍难点的《俺の妹がこんなに可愛いわけがない》，我猜大部分人应该勉强还能同意按照字符数将标题长度算成16；如果按字符数量计算的话，那么这个标题《All You Need Is Kill》的长度算是20是否合适，我认为可能已经有人会不同意了——觉得这里应该按单词个数算长度，即使算上空格也只能算9；假设按照前面的标准那么下个一个标题《ソードアート・オンライン》到底应该算12还是4，ソードアート・オンライン就是sword art online的片假写法；最后还有之前比较火的动画原作小说《Re:ゼロから始める異世界生活》，前面那个Re又到底怎么算？为了统一标（tou）准（lan），我决定就简单粗暴直接按照字符串长度计算标题长度。

解决完标题长度定义的问题后，又有一个问题需要认识到——那就是评论数对分数的影响，特别是低评论数量下分数的可信度问题，当评论数据比较少时，这个小说的评分可能是不准确的。

下面就开始展示以及解读分析结果

### Q1. 轻小说的标题是否随着年份的增加正在变得更长？

使用到的统计代码如下
{% highlight R linenos %}
# 加载作图工具与数据
library(ggplot2)
library(RMySQL)
con = dbConnect(MySQL(), host = "localhost", dbname = "bangumi")
rs = dbSendQuery(con, "select * from light_novels")
novels = dbFetch(rs, n = Inf)
novels$title_length = nchar(novels$title_jp)

# R代码
# 箱式图
ggplot(data = novels, aes(x = year, y = title_length, group = year)) +
  geom_boxplot() +
  xlab("年份") +
  ylab("标题长度（字符数）")

# 线性模型
year.lm = lm(title_length ~ c(year-2017), data = novels)
summary(year.lm)
anova(year.lm)
{% endhighlight %}

![pictrue](http://wx1.sinaimg.cn/mw690/6daafd01gy1fewrax2u4lj20dc0dc74e.jpg)
<center>图1. 不同年份轻小说标题长度的箱式图</center>


从图1中可以看出一些基本的规律，轻小说的标题长度范围是很大的，不过75%（即图中盒子的上边缘）的小说标题要少于30个字符，中位数（盒子中间的黑线）有微弱的上升。接下来我们建立一个年份到标题长度之间的线性模型，以期能更好更明确地回答Q1。

<center>表1. 出版年份对轻小说标题长度线性模型的回归结果</center>

|               |Estimate |Std. Error |t-value |Pr(>\|t\|)|
|Intercept      |13.08108 |0.06052    |216.15  |<2e-16 ***|
|c(year - 2007) |0.17265  |0.01070    |16.14   |<2e-16 ***|

回归前年份是处理成连续变量还是分类变量的区别是——如果你只是想知道各年份间是否具有差异，那么应该处理成分类变量，现在Q1的问题是标题是否逐年增加？因此应该处理为连续变量。另外由于年份的最小值是2007，回归前年份统一减去了2007。实际上年份是否统一减去2007只会影响截距的分析结果，对系数的回归结果不会有任何影响。那么这里为何要减去2007呢？如果你不减去2007,截距的值为-333.428。这个数据意思可以理解为——当year取0的时候，预测标题长度为-333.428个字符。当然，因为实际的数据是在2007-2016之间的，这个解读本身也不太正确，但是-333这个截距数据看起来不那么直观是显然。如果减去2007之后，获得的截距数据13.08直接对应为2007年的平均标题长度估计值，这看起来就直观多了。

表1告诉我们系数0.17265的p值是显著的，说明系数等于0.17265的可靠程度非常高。但这个值本身的大小也说明，随着年份增加，标题平均值变长的幅度非常小。方差分析能进一步证实我们的说法

<center>表2. 出版年份对轻小说标题长度线性模型的方差分析</center>

|               |Df    | Sum Sq | Mean Sq | F-value |   Pr(>F)     |    
|c(year - 2007) |     1|   14915| 14915.0 |  260.35 | < 2.2e-16 ***|
|Residuals      | 64972| 3722116|    57.3 | ||

表2中可以看出来，年份的方差虽然是显著的，但是只能解释标题方差中非常小的一部分。因此，关于Q1可以做出以下回答——**轻小说的标题的确随着年份的增加正在变得更长，但是变长的幅度很小，也就是说年份带来的影响非常小**

### Q2. 轻小说的标题长度与评分之间是否存在显著的负相关？
{% highlight R linenos %}
# R代码
# 筛选数据
rated_novels = subset(novels, comment >= 10)

# 最初的线性模型
rate.lm = lm(rate ~ title_length, data = rated_novels)
summary(rate.lm)

# 限制评论数多余60时的线性模型
rate.lm = lm(rate ~ title_length, data = rated_novels, subset = comment >= 60)
summary(rate.lm)

# 最小评论数逐渐上升时，标题长度系数变化
coef_title_len = function(min_comment){
    lm(rate ~ title_length, data = rated_novels, subset = comment >= min_comment)$coefficients['title_length']
}

coef_title_len_seq = sapply(seq(10,100,10), coef_title_len)
coef_df = data.frame(min_comment = seq(10,100,10), coef_title_len_seq = coef_title_len_seq)

ggplot(data = coef_df, aes(x = min_comment, y = coef_title_len_seq)) +
    geom_line() +
    xlab("最小评论数") +
    ylab("标题长度的回归系数")

# 权重回归
rate.lm = lm(rate ~ title_length, data = rated_novels, weight = comment)
summary(rate.lm)
anova(rate.lm)

{% endhighlight %}

<center>表3. 轻小说标题长度对评分线性模型的回归结果</center>

|             |Estimate |Std. Error |t value |Pr(>\|t\|)    |
|(Intercept)  |7.502798 |  0.027211 |275.728 |  <2e-16 ***|
|title_length |0.003877 |  0.001726 |  2.246 |  0.0248 * |

从表3的结果来看，标题长度与评分存在显著的正相关。那么我们是不是就可以终结Q2，并且做出标题越长评分越高的结论？不能着急，还有一个很重要方面，我们还没有涉及到，那就是评分的可靠性。例如：对数据做出进一步的限制——评论数大于60之后，我们得到了一个全新的结果（表4）。

<center>表4. 评论数多于60的轻小说标题长度对评分线性模型的回归结果</center>

|             |Estimate |Std. Error| t value |Pr(>|t|)    |
|(Intercept)  | 8.05397 |   0.05660| 142.306 | < 2e-16 ***|
|title_length |-0.01266 |   0.00396|  -3.197 | 0.00161 ** |

在表4中，系数变成了负数，结论完全颠倒了过来。实际上，当我们依次把10，20，30...100以上评论的线性回归结果汇总到一块，可以发现标题长度的回归系数会越来越小（见图2）。

![pictrue](http://wx2.sinaimg.cn/mw690/6daafd01gy1fewrb1dwbjj20dc0dc3yg.jpg)
<center>图2. 标题长度回归系数随评论阈值的变化</center>


直接使用上面分层数据回归的结果作为结论也是可行的，但是无法给出一个相对准确和统一的结论。原因在于无法确定一个评论数阈值，而且确定了某个评论数阈值之后，低于该阈值的数据在回归中实际上被舍弃了，这是一种对数据浪费。实际上，基于评论数越多数据越可靠的假设，使用评论数作为回归权重来对最初的模型进行调整，我们就可以获得一个相对可靠的结果。

<center>表5. 以评论数为权重的轻小说标题长度对评分线性模型的回归结果</center>

|             | Estimate| Std. Error| t value| Pr(>\|t\|)  |
|(Intercept)  | 7.800790|   0.023806| 327.687|  < 2e-16 ***|
|title_length |-0.006169|   0.001583|  -3.896| 0.000101 ***|

<center>表6. 以评论数为权重的轻小说标题长度对评分线性模型的方差分析</center>

|             |  Df | Sum Sq |Mean Sq |F value |   Pr(>F)    |
|title_length |   1 |  146.2 |146.225 | 15.179 |0.0001007 ***|
|Residuals    |2193 |21126.0 |  9.633 |||

表5和表6的数据使得我们对Q2可以给出一个相对可靠的结论。标题的系数是负数，说明标题越长，评分的期望值越低。另一方面，这个系数又是如此的小，即使标题长度达到了100个字符，评分的期望值也只会下降0.6分。方差分析的结果也显示出来，标题长度作为自变量只能解释不到0.7%评分变化。因此，关于Q2可以做出以下回答——**轻小说的标题长度与评分之间存在显著的负相关，然而这个负相关并没有什么用。因为这个变化幅度非常小，以致于读者很难察觉到。**
