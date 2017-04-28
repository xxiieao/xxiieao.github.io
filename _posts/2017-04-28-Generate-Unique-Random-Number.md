---
layout: post
title:  "Generate Unique Random Number"
date: 2017-04-28 20:04:12 +0800
categories: "algorithm"
author: xxiieao
---

这次的blog和之前的《RSA and Pseudo-random number》还有有些关系，由来是这样的。同事收到了这样一个需求，分批生成16位的随机数字，所有的随机数都要求唯一。使用RSA算法可以帮助我们实现这个功能，先假设输入为n，先需要找到两个素数p和q，p和q的乘积大概在10^17左右即可，而后再寻找到一个s，这个s和(p-1)\*(p+1)互质，最后使用公式

$$n^s mod (p*q)$$

计算即可。同事表示道理我都懂，合适大小的数字到底怎么寻找？

将思路整理一下，简单地来说我们需要三个函数

- 高次方求余函数
- 质数验证函数
- 互质验证函数

### 高次方数求余函数

这个东西说基本也基本，但是估计也有不一些人碰到过这个问题的。例如$$11^500 (mod 13)$$，如果你在python里用(11**500)%13是绝对不行的，因为数值太大可能会撑爆内存。但是如果稍微灵活一点处理这个问题，我们就想到

$$
11^{500} (mod 13) \\
\equiv 121 * 11^{498} (mod 13) \\
\equiv (9 * 13 + 4) * 11^{498} (mod 13) \\
\equiv 4 * 11^{498} (mod 13) \\
...
$$

根据上面的思路我们可以写出下面的函数

{% highlight python linenos %}
def mod(base, index, divisor, res=1):
    if(index == 0):
        return res
    else:
        return mod(base, index-1, divisor, (res * base) % divisor)

mod(11, 500, 13)
# 9
{% endhighlight %}

不过这个函数显然还是有效率上的缺陷的，index有多大就要迭代多少次。那么怎么对其进行改进呢？继续上面的公式推导，我们能得到

$$
11^{500} (mod 13) \\
\equiv 121^{250} (mod 13) \\
\equiv (9 * 13 + 4)^{250} (mod 13) \\
\equiv 4^{250} (mod 13) \\
...
$$

这里你会发现，index是按照对数衰减的。换句话说，我们把一个时间复杂度为$$O(n)$$的算法变成了一个$$O(log_2{n})$$的算法，那么按照上的思路我们有下面的代码

{% highlight python linenos %}
def quick_mod(base, index, divisor, res=1):
    if(index == 0):
        return res
    elif(index == 1):
        return (base * res) % divisor
    else:
        if(index % 2 == 1):
            res = (base * res) % divisor
        index = index / 2
        base = (base ** 2) % divisor
        return quick_mod(base, index, divisor, res)

quick_mod(11, 5000000, 987654321)
# 619359709
{% endhighlight %}

到此关于高次方数求余，我们获得了基本满意的算法

### 质数验证函数

我这次使用的算法（wilson定理）并不是效率很高的算法，之所以采取这个算法，主要是这次需求比较特殊，需要生成的质数不是很大同时只需要生成一次就可以永久使用。wilson定理说起非常简单，如果p为质数则有

$$(p-1)! (mod p) \equiv p - 1$$

根据这个特点我们可以写出下面的函数
{% highlight python linenos %}
def is_prime(number):
    res = 1
    iter = number - 1
    while(iter > 0):
        res = (res * iter) % number
        iter -= 1
    if(res == number - 1):
        return "%d is prime" % number
    else:
        return "%d is not prime" % number

is_prime(7)
# '7 is prime'
is_prime(8)
# '8 is not prime'
{% endhighlight %}

下一步我们需要考虑的是获得两个乘积在$$10^{17}$$左右的素数，其实这里我也采用较为暴力的方式解决，直接搜索99999959到100000059之间的奇数。

{% highlight python linenos %}
for number in range(99999959, 99999959 + 100, 2):
    is_prime(number)

# '99999959 is prime'
# '99999961 is not prime'
# ...
# '99999971 is prime'
# ...
# '99999989 is prime'
# ...
# '100000007 is prime'
# ...
# '100000037 is prime'
# '100000039 is prime'
# ...
# '100000049 is prime'
{% endhighlight %}

99999989和100000007是一对不错的数，乘积为9999999599999923。另外还发现质数比想象中密集好多，居然还发现了一对孪生素数100000037和100000039。下一个任务是找到多个和9999999399999928互质的数。

### 互质验证函数

所谓互质也就是两个不同的正整数之间没有公约数，互质验证也有很方便的算法。这次我们先上代码再谈谈证明

{% highlight python linenos %}
def coprime(number1, number2):
    if(number1 < number2):
        number1, number2 = number2, number1
    res = number1 % number2
    if(res == 1):
        return "coprime"
    elif(res == 0):
        return "not coprime"
    else:
        return coprime(number2, res)

coprime(11, 9999999399999928)
# 'not coprime'
coprime(13, 9999999399999928)
# 'coprime'
{% endhighlight %}

这个算法的名字其实叫做欧几里得辗转相除法。辗转相除的意思是——用任意两个不相等的正整数中较大数除以较小的数取余数，然后利用上一次的除数（即较小的数）除以刚刚取得的余数重复上面的过程。

设a和b是两个不相等的正整数，并且a比b大，令a和b的最大公约数表示为

$$ u = gcd(a, b)$$

此外显然有

$$ a = k * b + c $$

因为，u是a和b的最大公约数，那么我们可以得到

$$
a = s * u \\
b = t * u \\
c = (s - k * t) * u
$$

换句话说c也能被a和b之间的最大公约数整除，那么也就是说u也是b和c之间的最大公约数。即下面公式成立

$$ gcd(a, b) = gcd(b, c) $$

这里也许有人要质疑一下，也许t和(s - k * t)之间有大于1的公约数，那么gcd(b, c)就会比gcd(a, b)要大了。这是不可能的，为什么呢？因为u既然是a和b的最大公约数那就说明，s和t的最大公约数为1。因为如果gcd(s,t)不为1，那么u就是不是a和b的最大公约数了。既然s和t的最大公约数为1，那么t和(s - k * t)之间的最大公约数也是1。也就是说我们可以通过不断地求余来迅速寻找两个数之间的最大公约数，显然如果两个数互质那么他们的最大公约数就是1。

至此三个函数都已经写好，剩下的问题也都能迎刃而解了。
