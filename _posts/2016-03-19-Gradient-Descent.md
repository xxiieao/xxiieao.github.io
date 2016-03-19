---
layout: post
title:  "Gradient Descent"
date: 2016-03-19 10:03:50 +0800
categories: "algorithm"
author: xxiieao
---

梯度下降法（Gradient Descent）是机器学习中一个比较核心的内容，另一核心的内容是代价函数（cost function）。这两个内容，前者负责指导机器如何进行学习，后者负责评定机器学习的效果。机器学习本质上来说是一种求最优解的过程。这个过程中，需要优化的就是代价函数，而梯度下降则是我们达到最优解的方法。要理解梯度下降法，我们先来看一看它与普通方程求解有什么不同。

先假设我们在平面上有10个散点，我们的目标是求散点的回归线。回归线的求解，在许多统计课上都被重复过无数遍的经典方法——最小二乘法。最小二乘法通过偏微分求导获得最优解，具体的求解过程就不展开了，让R代码代劳吧。

{% highlight R linenos %}
x_values = 1:10
y_values = 1:10 + runif(10) # 由于这里是随机函数，你看到图形可能和我不一样

linear_reg = lm(y_values ~ x_values)
summary(linear_reg)

# R的返回结果
# Call:
# lm(formula = y_values ~ x_values)
#
# Residuals:
#      Min       1Q   Median       3Q      Max
# -0.52199 -0.24376  0.07777  0.28978  0.35702
#
# Coefficients:
#             Estimate Std. Error t value Pr(>|t|)    
# (Intercept)  0.27201    0.24963    1.09    0.308    
# x_values     1.04593    0.04023   26.00 5.14e-09 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
#
# Residual standard error: 0.3654 on 8 degrees of freedom
# Multiple R-squared:  0.9883,	Adjusted R-squared:  0.9868
# F-statistic: 675.9 on 1 and 8 DF,  p-value: 5.144e-09
#

# 图形展示
plot(x_values, y_values)
abline(linear_reg, lty = 2)
legend(2, 10, c('data', 'regression line'), pch = c(1, NA), lty = c(NA, 2), bty = 'n')

{% endhighlight %}

大致效果如图
![picture](http://ww3.sinaimg.cn/mw690/6daafd01gw1f2239zk6rmj20dc0dcq34.jpg)

那么梯度下降法是怎么做的呢？
{% highlight R linenos %}
# 准备工作，建立模型函数
linear_function = function(x, params){
  intercept = params[1]
  coef = params[2]
  x * coef + intercept
}

# 第一步，首先建立代价函数。线性回归通常使用预测值与实际值之间的方差做为代价函数。
cost_function = function(y_prediction, y_real){
  mean((y_real - y_prediction)^2)
}

# 第二步，决定梯度下降的学习函数。
learning_function = function(params, x_real, y_real, learning_rate){
  intercept_delta = mean(2 * (y_real - linear_function(x_real, params)))
  coef_delta = mean(2 * (y_real - linear_function(x_real, params)) * x_real)
  c(params[1] + learning_rate * intercept_delta, params[2] + learning_rate * coef_delta)
}

# 第三步，决定学习效率。
lr = 0.001 # 通常为0.01～0.02，为了让学习曲线更清晰，这里选取比较小的0.001

# 第四步，设置停止条件，开始迭代学习。
# 本例中是迭代200次
# 需要注意的是，在本例子中由于x和y的数值都比较小，没有进行标准化。
# 如果变量特别大或者特别小必须进行标准化。

init_params = c(0, 0)
cost_dynamic = NULL

for(i in 1:200){
  cost = cost_function(linear_function(x_values, init_params), y_values)
  init_params = learning_function(init_params, x_values, y_values, lr)
  cost_dynamic[i] = cost
  params_dynamic$intercept[i+1] = init_params[1]
  params_dynamic$coef[i+1] = init_params[2]
}

init_params
# [1] 0.1623132 1.0616849
# 这两个参数虽然并不是最优解，不过也非常接近了

# 代价函数的变化过程
plot(1:200, cost_dynamic, typ = 'l')

{% endhighlight %}

cost_function值在迭代中的变化
![picture](http://ww4.sinaimg.cn/mw690/6daafd01gw1f2239zmfclj20dc0dc0st.jpg)

我们可以发现，与最小二乘法不一样，梯度下降法并不能精确地获得最优解，很多时候获得的是最优解的近似解。那么在实际应用中，我们明明有类似直接通过代数方法求解最优解的方法，为何还要使用梯度下降法呢？因为在实际求解中，有些代数方法计算量可能会非常巨大，例如高维矩阵求逆，或者某些复杂的代价函数导致根本无法进行代数求解。在这种情况下，梯度下降法就展现出其作用来了。
