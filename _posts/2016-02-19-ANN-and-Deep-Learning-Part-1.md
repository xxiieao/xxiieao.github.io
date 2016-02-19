---
layout: post
title:  "ANN and Deep Learning Part 1"
date: 2016-02-19 22:02:11
categories: "algorithm"
author: xxiieao
---

随着Google的围棋人工智能AlphaGo战胜欧洲冠军樊麾，深度学习开始吸引大众的目光。已经有部分媒体开始打起了嘴仗，话题从李世石的胜率一直吵到AlphaGo是不是被过度炒作，樊麾是不是有放水。但是很多人显然没有意识到，AlphaGo的成功只不过是又在一个领域证明了深度学习的强大能力而已。在围棋之外，深度学习早已在模式识别等领域站稳了脚跟。对于我个人而言，如果某一天AlphaGo在围棋领域上战胜了人类的顶尖高手，这只能说明人类思考的过程可能远比我们想象的要简单。
在我开始了解深度学习之前，先需要解决的一个问题是什么是深度学习，深度学习与人工神经网络有什么关系？
人工神经网络（ANN，Artificial Neural Network）是机器学习中一个重要的类别，其实际上是指一类模拟人类神经信号传递的统计学习算法。深度学习则是模仿的人类视觉机理。咋一看，二者非常相像啊，特别是二者都有一个网状的结构。实际上两者有着非常大的差别，主要体现在二者的设计哲学上。
人工神经网络是主要通过调整神经元的活跃与否来实现逼近需要学习的结果的，理论上来说如果网络中的隐含层数越多，人工神经网络模拟非线性函数的效果应该越好。但实际上由于学习算法的缺陷，导致多层网络结构非常难以实现。以反向传播算法为例，误差在反向传播的过程，由于每层的稀释，传播到输入层时，剩余误差已经非常小，根本不能对网络权重进行有效的调节。因此，很多实践中的人工神经网络只有一层隐含层。
而深度学习则采用了完全不同的设计哲学，将输入的信号在每一层的处理看作是一次对信号的抽象过程。因此深度学习实际上就是逐层处理，逐层抽象的过程。这一设计类似于人类首先建立简单的概念，然后通过组合这些简单的概念来构建更复杂的概念。正因为如此，有人评价深度学习充分挖掘了人工神经网络的潜力。
关于人工神经网络和深度学习更详细的内容，显然不能光凭借上面简单的句子就能描述清楚，下一个部分我会先介绍人工神经网络的原理以及相关算法。