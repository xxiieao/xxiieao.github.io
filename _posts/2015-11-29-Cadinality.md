---
layout: post
title:  "Cadinality"
date: 2015-01-24 14:11:34
categories: "algorithm"
author: xxiieao
---

基数统计的ruby模拟代码，之所以说是模拟，因为bit操作基本上是利用2的幂运算来模拟的，而bitmap也是用数组来模拟的，所以内存使用什么大家就不要吐槽啦。

先假设有m个元素需要统计，并且哈希函数将输入一个字符串转换为一个在$$[0, 2^{32}-1]$$之间的整数（另外$$2^{32}$$可以估计大约24亿个元素，如果不够还可以换64，128）：

普通方法，空间消耗就不用说了，肯定是非常大，其次是计算其时间复杂度是$$O(m!)$$（阶乘是比次方还可怕的存在）

bitmap方法，空间消耗 m bit，但是时间复杂度是$$O(m)$$。

probabilistic counting一条bitmap使用的内存理论上为$$log_{2}(m)$$bit，但是在例子中经过hash之后必须则要有33bit。如果像例子中一样用16条来估算的话你可能需要一个float也就是另外32bit来记录每个bitmap的结果以便估算平均值，因此是33bit + 32bit。计算次数是bitmap数量乘以m，因此时间复杂度还是在$$O(m)$$。

Loglog和Hyploglog的空间复杂度都和桶数有关。像例子一样使用$$2^{10}$$个桶，由于每个桶只需要记录最大数字就行，而去掉前10位之后，最大值位于$$[0,22]$$之内，因此每个桶的最大值都需要5bit。最终大约需要$$2^{10}*5 = 640$$byte。时间复杂度仍然是$$O(m)$$。loglog的得名就是因为每个分桶需要的空间复杂度是$$log_2(log_2(m))$$。

{% highlight Ruby linenos %}
#需要的哈希函数gem
require 'murmurhash3'

#方便计算的方法
class String
  def hash(seed = 1234567) #输入一个字符串将其转换为一个在[0, 2^32-1]之间的整数
    MurmurHash3::V32.str_hash(self, seed)
  end
end

class Array
  def mean #输入一个数字数组，输出其平均值
    self.inject(0){|s, e| s += e} / self.length.to_f
  end
end

class Fixnum
#输入一个数字返回这个数字二进制形式下零前缀的个数
  def prf_zero_count(start = 31) 
    pos = 0
    start.step(0, -1).each{|power| (self >= (2^power)) ? break : pos += 1}
    pos
  end

#输入一个数字，返回两个值
#第一个值是这个数字二进制形式前power位数字，作为桶编号
#第二个值是剩下32 - power位二进制数字中零前缀个数
  def bucket(power)
    buck, resd =  self.divmod(2^32 / 2^power)
    [buck, resd.prf_zero_count(31 - power)]
  end

end

#基数统计方法类
class Cadinality
attr_reader :bitmap

  def initialize(str_list)
    @str_list = str_list
  end

  def prob_count(bitmap_num)
    ave_index = 0#这个@bitmap里实际上记录的是bitmap_num次prob_count
    0.step(bitmap_num - 1).each do |i| #下面的每一次循环都是一次独立的prob_count
      @bitmap = Array.new(33, 0) #bitmap其二进制形式一共有33位，这里用一个33维数组来模拟
      hash_seed = (rand() * 100000).round #为每一次prob_count重新分配一个hash种子
      @str_list.each{|str| bitmap[str.hash(hash_seed).prf_zero_count] = 1}
      #将str_list里面的每一元素hash化，统计零前缀的个数，然后将对应bitmap位置的元素从0改为1
      ave_index += bitmap.index(0)
      #求第一个bitmap中第一个0出现的位置，并且将其计入ave_index中以便后面求平均值
    end
    2 ^ (ave_index/bitmap_num) / 0.77351
  end

  def loglog(buk_ind)
    buk_num = 2 ^ buk_ind #根据输入buk_ind计算分桶数
    @bitmap = Array.new(buk_num, 0) #桶内数值初始化为0
    @str_list.each do |str|
      bucket_no, zerobitcount = str.hash.bucket(buk_ind) #计算str_list每一个元素其分桶编号和零前缀个数
      bitmap[bucket_no] = zerobitcount if zerobitcount > bitmap[bucket_no] #每个分桶只记录最长的0前缀数量
    end
    2 ^ bitmap.mean * buk_num * 0.794
  end

  def hyperloglog(buk_ind)
    buk_num = 2 ^ buk_ind #根据输入buk_ind计算分桶数
    @bitmap = Array.new(buk_num, 0) #桶内数值初始化为0
    @str_list.each do |str|
      bucket_no, zerobitcount = str.hash.bucket(buk_ind) #计算str_list每一个元素其分桶编号和零前缀个数
      bitmap[bucket_no] = zerobitcount if zerobitcount > bitmap[bucket_no] #每个分桶只记录最长的0前缀数量
    end
    (0.7213/(1+1.079/buk_num)) * (buk_num^2) / bitmap.inject(0){|s, e| s += 2 ^ - (e + 1)}
    #值得注意的一点hyperloglog方法使用的0前缀第一个算1因此相对前两种要+1
  end

end

samples = Array.new(500000) { |e| e += 1 } #生成一个1-50w的数组
samples.shuffle! #随机打乱位置
samples = samples.map{|e| e.to_s} #转换成字符方便使用哈希函数
cadinality = Cadinality.new(samples[0+10000..99999+10000]) #取前面的10w个

cadinality.prob_count(16) / 100000 -1 #使用16条bitmap统计时probabilistic counting的误差
cadinality.loglog(10) / 100000 -1          #使用2^10个分桶loglog的误差
cadinality.hyperloglog(10) / 100000 -1 #使用2^10个分桶hyperloglog的误差
{% endhighlight %}