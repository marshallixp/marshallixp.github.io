---
layout: post
title:  "Train on Pascal(2)"
date:   2016-07-27 15:09:00
categories: segmentation
---

由于前一次训练中发现网络并没有收敛，经过查阅资料，我们决定采用以下调整方案：  
1) Reduce learning rate by a factor of 10  
2) Normalize softmax loss (divide with the number of pixels -> normalize: true)  
3) Try net surgery  
4) mean RGB / BGR ?  

**要点：**

a) Make sure you do not make the model to learn from the background (ignore_label=255)  
b) IU score / pixel-wise accuracy are more important
c) Check each layer to see it's initialized with correct values
   
   

**Fcn32s Training**  

根据调整方案1），目前采用fixed的学习率，取值1e-9。根据2），将softmaxloss层的normalize设置为true。根据修改后的参数模型，经过4000次迭代以后，我们发现loss是在不断减小，说明模型可以收敛，但是主要的指标参数依然没有改变，需要等待进一步的结果。  

    >>> 2016-07-27 15:58:21.062000 Iteration 4000 loss 3.04448500594  
    >>> 2016-07-27 15:58:21.062184 Iteration 4000 overall accuracy 0.733180213097  
    >>> 2016-07-27 15:58:21.062357 Iteration 4000 mean accuracy 0.047619047619  
    >>> 2016-07-27 15:58:21.062504 Iteration 4000 mean IU 0.0349133434808  
    >>> 2016-07-27 15:58:21.062591 Iteration 4000 fwavacc 0.537553224877  

当从32000次迭代结果恢复以后，我们发现网络的损失是以很小的数值下降，约为3e-5的减小值。但是，主要指标依然没有变化。  

    >>> 2016-07-28 12:56:20.636460 Begin seg tests
    >>> 2016-07-28 12:59:02.790584 Iteration 48000 loss 3.04407284242
    >>> 2016-07-28 12:59:02.790747 Iteration 48000 overall accuracy 0.733180213097
    >>> 2016-07-28 12:59:02.790932 Iteration 48000 mean accuracy 0.047619047619
    >>> 2016-07-28 12:59:02.791069 Iteration 48000 mean IU 0.0349133434808
    >>> 2016-07-28 12:59:02.791161 Iteration 48000 fwavacc 0.537553224877
    >>> 2016-07-28 13:21:27.470571 Begin seg tests
    >>> 2016-07-28 13:24:08.706496 Iteration 52000 loss 3.04403515862
    >>> 2016-07-28 13:24:08.706656 Iteration 52000 overall accuracy 0.733180213097
    >>> 2016-07-28 13:24:08.706831 Iteration 52000 mean accuracy 0.047619047619
    >>> 2016-07-28 13:24:08.706970 Iteration 52000 mean IU 0.0349133434808
    >>> 2016-07-28 13:24:08.707063 Iteration 52000 fwavacc 0.537553224877  
对网络层进行检查，我们很遗憾地发现fc6_conv, fc7_conv, score_fr层，并没有被初始化，也没有训练更新，这是目前网络无法收敛的原因。

根据上述分析，采取如下调整：
1) vgg16相关的层冻结，对fc6_conv, fc7_conv, score_fr, upscore激活学习参数,并且初始化，如下：  
param {
    lr_mult: 1
    decay_mult: 1
}  
param {
    lr_mult: 2
    decay_mult: 0
}  
convolution_param {
    num_output: 4096
    pad: 0
    kernel_size: 1
    stride: 1
    weight_filler {
      type: "gaussian"
      std: 0.01
    }
    bias_filler {
      type: "constant"
      value: 0.1
    }
}  

2）采用fixed的学习率1e-10，monentum为0.99。在经过200次迭代之后，我们观察相关层的变化情况。然而，发现除了初始化成功外，依然没有进行训练，即相关层的权重几乎没有改变（1000次迭代，除了score_fr层有微小改变）。  

考虑到可能是学习率设置太小，现将学习率改为1e-6, momentum为0.9，再次进行训练。发现果然如预期，除了固定权重层（vgg对应）以外，其余各层权重均开始改变。设想训练参数及网络模型有效，需要进行再次训练验证。同时，根据文章中的参数设置，我们重新设置训练主要参数如下：  
lr_policy: "fixed"  
base_lr: 1e-4  
momentum: 0.9  
weight_decay: 0.0005  
于是，4000次迭代之后的结果如下：

    >>> 2016-07-28 15:40:30.566007 Iteration 4000 loss 0.778730130796
    >>> 2016-07-28 15:40:30.566297 Iteration 4000 overall accuracy 0.833765982794
    >>> 2016-07-28 15:40:30.566349 Iteration 4000 mean accuracy 0.654612851421
    >>> 2016-07-28 15:40:30.566486 Iteration 4000 mean IU 0.450926431181
    >>> 2016-07-28 15:40:30.566580 Iteration 4000 fwavacc 0.742218942131
可见，最终主要指标都出现大幅提升，说明训练有效！
  
  
**SegNet Training**  

根据调整方案1)，采用fixed的学习率1e-10，monentum为0.99。根据2），将softmaxloss层的normalize设置为true。  
同样，经过4000次迭代以后，我们发现修改后的参数模型，loss可以不断减小，即表示可以收敛，而且收敛速度快于Fcn32s网络，但是最主要的指标参数也依然没有出现变化，需等待进一步训练结果。  

    >>> 2016-07-27 16:36:01.449254 Iteration 4000 loss 3.04294351251  
    >>> 2016-07-27 16:36:01.449524 Iteration 4000 overall accuracy 0.733180213097  
    >>> 2016-07-27 16:36:01.449573 Iteration 4000 mean accuracy 0.047619047619  
    >>> 2016-07-27 16:36:01.449706 Iteration 4000 mean IU 0.0349133434808  
    >>> 2016-07-27 16:36:01.449798 Iteration 4000 fwavacc 0.537553224877  
由于之前的Fcn32s网络实现了有效训练，我们将相关调整也作用于SegNet网络上，测试结果。
  
  
**小结**

由于一开始并没有弄清楚各个训练参数和网络模型参数的意思，始终将vgg16对应层进入训练，相反最后的score_fr和upscore层并没有进行训练;除此以外，除vgg16对应层外的如fc6_conv等并没有有效初始化，无法实现参数更新。而且，当训练参数如base_lr设置太小（如1e-10）也会导致训练收敛速度过慢。当然，normalize softmax loss也是必须的。
