---
layout: post
title:  "Tricks in Plotting"
date: 2016-07-09 19:07:28 +0800
categories: "math"
author: xxiieao
---

数据作图其实是蛮有技巧和讲究的一项技术，往大了说其实就是数据可视化。不过今天不打算谈数据可视化这么大的题目，何况我对这方面也不是很了解。今天要谈的就是最普通数据的作图，没错就是选中Excel里面数据列，那个点一下图表按钮就能出来那个东西。

不过在讨论作图本身之前，首先需要解决一个问题为什么要作图？显而易见，图片是为了传达信息的。那么另一种常用的数据展示形式——表格和图片又有什么区别呢？图片比起表格在某些条件下要更加直观。比如说，12个月的月销售数据，你需要把十二个数据扫视一遍才能找到哪个月份最高，而如果是图表你可能只需要瞄一眼就可以了。另外更重要的一点是，图片比表格要美观得多。现在是一个数据处理高速发展的时代，各种作图工具也是层出不穷，做出来的图片也是一个比一个要美观。不过，在追求美观的道路上我感觉有相当一部分人已经走在歪路之上。虽然，图片的美观是非常重要的，但是图片需要传递的信息才是最为根本的，尽力保证数据不失真更是重中之重。下面我谈到两种我遇到过比较典型的作图错误：

#### 过分简洁

自从苹果发布会上，乔帮主用了那个蓝底白字的PPT后，电子科技界似乎完全迷上了这种简洁主义。我曾经参加过一次大数据峰会，在这个峰会上有许多非常有分量的公司分享了在大数据应用方面的经验。期间，自然免不了展示一些数据。令我有些吃惊的是，这些专业的公司竟然有时候会出现类似下面的图片。

![pictrue](http://ww4.sinaimg.cn/mw690/6daafd01gw1f5sjyf0wz1j20dc0dc3yq.jpg)

这个图简洁是简洁了，但是想要传达的信息也没了。可以看到我举例的这张图片缺少了坐标轴，标题和图例，我相信当任何一个观众单独看到这样一张图片的时候，他无法接收到任何有效信息。对于图片而言有一个原则就是任何一张图片都应该是一个不言自明的故事。因此下图才是一个相对完整的故事。

![pictrue](http://ww3.sinaimg.cn/mw690/6daafd01gw1f5sjyjqtm2j20dc0dcjru.jpg)

{% highlight R linenos %}
# R代码
item_sales_a <- c(200, 300, 250, 240)
item_sales_b <- c(180, 280, 270, 250)
months <- 1:4

# 错误的作图
plot(0, 0, xlim = c(1, 4), ylim = c(150, 350), typ = 'n', bty = 'n', xlab = '', ylab = '', xaxt = 'n', yaxt = 'n')
lines(y = item_sales_a, x = months, col = 'red')
lines(y = item_sales_b, x = months, col = 'blue')
axis(1, labels = F)
axis(2, labels = F)

# 正确的作图
plot(y = item_sales_a, x = months, ylim = c(150, 350), typ = 'l', col = 'red', xaxt = 'n',
     main = 'Sales of First Season', xlab =  'montth', ylab = 'quantity')
lines(y = item_sales_b, x = months, col = 'blue')
axis(1, at = 1:4, labels = 1:4)
legend(x = 3.2, y = 350, c('item A', 'item B'), lty = 1, col = c('red', 'blue'), bty = 'n')
{% endhighlight %}

#### 滥用饼图

当我刚进入校园不久就受到过来自导师的告诫——“不要使用饼图”，奇怪的是商业上使用饼图似乎是家常便饭。为什么科研界不喜欢饼图呢，因为几乎所有饼图能做的事情，柱形图都能做，而且往往能做得更好。我举个简单的例子，例如一个男女比例数据。

![pictrue](http://ww1.sinaimg.cn/mw690/6daafd01gw1f5sjyof15fj20dc0dc3ym.jpg)

在饼图和柱形图都能够发现男性比较多，女性比较少。但是你很难直接从上面的饼图中准确估算男性比例比女性多多少，而在下面柱形图中完全可以做到。关于这个有一个解决方案是在饼图中加入一个数值表示各自的比例，不过如果你需要画的数据超过10个以上，如何标注这些数值就变成了一个灾难，而在柱形图中，y轴已经完美解决你的需求。

![pictrue](http://ww4.sinaimg.cn/mw690/6daafd01gw1f5sjyrg016j20dc0dc74f.jpg)

{% highlight R linenos %}
# R代码
gender_count <- c(450, 380)
names(gender_count) <- c('male', 'female')

# 饼图
pie(gender_count, main = 'Gender Ratio', init.angle = 90, clockwise = T)

# 柱状图
par(mar = c(5.1, 4.1, 4.1, 4.1))
barplot(gender_count, ylim = c(0, 500), main = 'Gender Ratio', ylab = 'count')
axis(4, at = seq(0, 0.6, 0.1) * sum(gender_count), labels = seq(0, 0.6, 0.1))
mtext('ratio', 4, 2.2)
{% endhighlight %}

实际上我认为，饼图可以用在一些对数值大小分辨要求不高，只需要传达大致比例或者排位的场合，因为饼图那个最终合成一个圆的设计的确比柱形图要好看。不过仍然需要注意的是不能干扰到图片最重要的原则，传递准确和有用的信息。典型失败的例子如下：

![pictrue](http://ww4.sinaimg.cn/mw690/6daafd01gw1f5sjyuwqblj20dc0dcgmw.jpg)

上面一张关于浏览器占比历年变化的图，作图者在这里选用了一种变形的饼图，或者我应该叫他环图？显然，这个选择是非常失败的，没有人能一眼能准确看出来哪种浏览器的比例在上升，哪种在下降，因为外圈环比内圈大的原因，他们看起来都在变大。显然用条形图来做这个图会清晰明了得多，而且分组条形图比堆叠条形图更加清晰。

![pictrue](http://ww1.sinaimg.cn/mw690/6daafd01gw1f5sjzehrg6j20dc0dc74y.jpg)

![pictrue](http://ww4.sinaimg.cn/mw690/6daafd01gw1f5sjzht6d4j20dc0dc3z4.jpg)

PS：最后的补充一下，用ggplot的时候忘记将因子排序的功能关掉，导致月份排序有些问题，但是图片已经上传懒得重排了，留着以后有心情再说吧。

{% highlight R linenos %}
# R代码
browser_ratio = data.frame(months = c('Feb', 'Mar', 'Apr', 'May'),
                           firefox = c(21.74, 21.80, 21.63, 21.71),
                           chrome = c(10.93, 11.57, 11.94, 12.52),
                           ie = c(56.77, 55.93, 55.11, 54.28),
                           opera = c(1.12, 1.06, 1.11, 1.27),
                           safari = c(5.76, 6.00, 6.51, 6.54),
                           other = c(3.68, 3.64, 3.70, 3.68))

browser_ratio
#   months firefox chrome    ie opera safari other
# 1    Feb   21.74  10.93 56.77  1.12   5.76  3.68
# 2    Mar   21.80  11.57 55.93  1.06   6.00  3.64
# 3    Apr   21.63  11.94 55.11  1.11   6.51  3.70
# 4    May   21.71  12.52 54.28  1.27   6.54  3.68

# 多重环形图有点复杂，我们需要动用大杀器ggplot2了
library(ggplot2)
library(reshape2)

browser_ratio_reshape = melt(browser_ratio, id = 'months')

browser_ratio_reshape
#    months variable value
# 1     Feb  firefox 21.74
# 2     Mar  firefox 21.80
# 3     Apr  firefox 21.63
# 4     May  firefox 21.71
# 5     Feb   chrome 10.93
# 6     Mar   chrome 11.57
# 7     Apr   chrome 11.94
# 8     May   chrome 12.52
# 9     Feb       ie 56.77
# 10    Mar       ie 55.93
# 11    Apr       ie 55.11
# 12    May       ie 54.28
# 13    Feb    opera  1.12
# 14    Mar    opera  1.06
# 15    Apr    opera  1.11
# 16    May    opera  1.27
# 17    Feb   safari  5.76
# 18    Mar   safari  6.00
# 19    Apr   safari  6.51
# 20    May   safari  6.54
# 21    Feb    other  3.68
# 22    Mar    other  3.64
# 23    Apr    other  3.70
# 24    May    other  3.68

ggplot(data = browser_ratio_reshape, aes(x = months, y = value / 100, fill = variable)) +
  geom_bar(stat = "identity") +
  scale_colour_brewer(palette = 'Set2') +
  labs(title = "Browser Market Ratio Variation in 2011", x = "Month", y = "Ratio", fill = "Browser") +
  coord_polar(theta="y")

ggplot(data = browser_ratio_reshape, aes(x = months, y = value / 100, fill = variable)) +
  geom_bar(stat = "identity") +
  scale_fill_brewer(palette = 'Set2') +
  labs(title = "Browser Market Ratio Variation in 2011", x = "Month", y = "Ratio", fill = "Browser")

ggplot(data = browser_ratio_reshape, aes(x = months, y = value / 100, fill = variable)) +
  geom_bar(stat = "identity", position="dodge") +
  scale_fill_brewer(palette = 'Set2') +
  labs(title = "Browser Market Ratio Variation in 2011", x = "Month", y = "Ratio", fill = "Browser")
{% endhighlight %}
