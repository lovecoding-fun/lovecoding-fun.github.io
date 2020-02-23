---
title: Paper Reading 2020-02-23
catalog: true
date: 2020-02-23 20:48:34
subtitle:
header-img:
mathjax: true
tags: ML
---

两篇论文的记录
### Scaling Up Neural Architecture Search with Big Single-Stage Models
Universal slimmable network的kernel+depth+resolution扩展版，方法上没啥创新，所以被拒了  
方法上就是Universal slimmable network那一套，然后加上了学习率和初始化的trick，之后要train这种包含多个子网络的网络可以借鉴一下里面的这些trick吧

Paper:[Scaling Up Neural Architecture Search with Big Single-Stage Models](https://openreview.net/forum?id=HJe7unNFDH)  
Codes:未开源

### Once for All: Train One Network and Specialize it for Efficient Deployment 
和上一篇想法差不多，只是做的比较细，方法上看起来有点东西不是那么糙。

![progressive shrinking](progressive_shrinking.png)
训练的时候提出了一个progressive shrinking的方法，先训好最大的网络，然后一步一步从kernel，depth和width进行elastic训练，中间kernel使用了一个transformation matrix，width用了channel sortting.

结果方面，最终效果很好就对了，finetune以后效果还能继续提高
![results](ofa-results1.png)

Paper:[Once for All: Train One Network and Specialize it for Efficient Deployment ](https://openreview.net/forum?id=HylxE1HKwS)  
Codes:[github](https://github.com/mit-han-lab/once-for-all)  
总的来看，其实也没有很创新的方法，只是看起来比上一篇细致一些然后实验上也充分，但实现上感觉还是很tricky不知道有多少水分。代码上感觉没有把完整过程开源出来，需要多机多卡的话复现也是个大麻烦，issue也不回，只能再观望一下看看怎么往detection走。

