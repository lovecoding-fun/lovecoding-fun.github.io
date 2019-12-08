---
title: What is Gumbel-Softmax Trick？
catalog: true
date: 2019-07-16 20:02:54
subtitle:
header-img:
mathjax: true
tags: ML
---

Gumbel-Softmax trick 在VAE、GAN、NAS等深度学习领域经常出现，那它到底是啥，有什么用呢？  

### 背景问题
首先简要介绍一下问题的背景。假设我们在网络中有一个有 \\(n\\) 个取值的离散分布\\(P(x)\\)，也就是一个\\(n\\)维的向量，每一个元素就是一个概率值。如果我们只是要对其进行采样或者得到最终概率最大的一个取值，我们很容易就可以直接采样或取\\(argmax\\)得到结果。但是，如果我们希望这个过程是可导的，该怎么办呢？

### Re-Parameterization Trick
首先介绍一个叫重参数化(Re-Parameterization)的东西，Re-Parameterization解决了上面提到的采样不可导的问题。这里用油管上一个[变分自动编码器教程](https://www.youtube.com/watch?v=9zKuYvjFFS8)中的图片来解释。
![re-paramerise](re-paramerise.png)
首先左边是重参数化之前的形式，我们希望通过学习到了标准差\\(\phi\\)和均值\\(x\\)，通过对分布\\(q\\)进行采样得到\\(z\\)，然后传入下一层网络。然而这就有个问题，采样过程是没法反传的，这样加载网络中就没法学习了，因此就有了右边的形式。  

在这里呢，其实就是在网络中只保留可以求导的部分，把不可导的采样部分放到网络外用一个新的输入节点\\(\epsilon\\)表示。\\(\epsilon\\)采样的过程和网络学习是分离的，因此不影响网络反传。  

具体而言，假设我们需要根据均值和方差从正太分布\\(N(x,\phi^2)\\)采样，我们可以转化为先从\\(\epsilon=N(0,1)\\)采样，然后计算\\(z=x+\epsilon\cdot \phi\\)来得到我们需要的\\(z\\)。

### Gumbel-Softmax Trick
通过Re-Parameterization Trick我们可以一定条件下解决采样不可导的问题。但回到我们最初的问题，我们并不是在连续的高斯分布上采样，而是在一个离散分布\\(P(x)\\)上采样。上面的重参数后是可导的，而换成离散分布以后，就无法满足了。所以，对于离散分布要怎么处理呢？
类似与前面的重参数技巧，我们对于离散变量有Gumbel-Max trick的采样方法：
![gumbel-max](gumbel1.png)
从上图中可以看出，我们将离散变量\\(\log{\alpha_1}, \log{\alpha_2}, \log{\alpha_3}\\)加上Gumbel噪声，然后对相加后的随机变量取\\(\argmax\\)后得到一个onehot向量，就是我们的采样结果，可以表示为：
$$x_{\alpha}=\arg\max(\log(\alpha_i)+G_i)$$
但\\(\argmax\\)这个操作是没法求导的，那么我们就用softmax来对\\(\argmax\\)进行松弛，这就得到了我们的Gumbel-Softmax Trick:
![gumbel-back](gumbel2.png)
其中，\\(\lambda\\)是softmax函数的temperature参数，用于控制采样 当\\(\lambda \to \infty\\)时，所有的激活值对应的激活概率趋近于相同（激活概率差异性较小）；而当\\(\lambda \to 0\\)，不同的激活值对应的激活概率差异也就越大，也就越趋向于onehot向量。（温度这个词主要来自于物理学中的温度，当温度高的时候，分子的运动就越剧烈，随机性就越大，反之则越稳定，随机性越小）  
到这里，我们既可以对分布进行采样，又能够满足在网络中可以反向传播学习。  
但我们可能还会有这样的问题：为什么不直接就用softmax函数呢，这样不是也能在网络中学习么？我们再回顾下我们的问题就能发现，我们需要采样+能反传，softmax满足能反传但是它不能达到我们根据概率分布采样的目的，而Gumbel-Max Trick就是用来近似分布然后采样的。  
另外，我们在此用到了Gumbel Distribution--\\(G\\)，是一个极值分布，通俗来讲就是分布的极值的分布，具体可参考：
[Gumbel distribution](https://en.wikipedia.org/wiki/Gumbel_distribution)

### 参考
[Gumbel-Softmax Trick和Gumbel分布](https://www.cnblogs.com/initial-h/p/9468974.html)
[The Gumbel-Softmax Trick for Inference of Discrete Variables](https://casmls.github.io/general/2017/02/01/GumbelSoftmax.html)
[Variational Autoencoders](https://www.youtube.com/watch?v=9zKuYvjFFS8)
[带你认识神奇的Gumbel trick](https://blog.csdn.net/a358463121/article/details/80820878)
[Talk: Categorical Reparameterization with Gumbel-Softmax & The Concrete Distribution](https://www.youtube.com/watch?v=wVkLM2KKHp8)