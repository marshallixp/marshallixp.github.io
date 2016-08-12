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
于是，结果如下：

    >>> 2016-07-28 15:37:43.526054 Begin seg tests
    >>> 2016-07-28 15:40:30.566007 Iteration 4000 loss 0.778730130796
    >>> 2016-07-28 15:40:30.566297 Iteration 4000 overall accuracy 0.833765982794
    >>> 2016-07-28 15:40:30.566349 Iteration 4000 mean accuracy 0.654612851421
    >>> 2016-07-28 15:40:30.566486 Iteration 4000 mean IU 0.450926431181
    >>> 2016-07-28 15:40:30.566580 Iteration 4000 fwavacc 0.742218942131
    >>> 2016-07-28 15:53:21.709648 Begin seg tests
    >>> 2016-07-28 15:56:08.182363 Iteration 8000 loss 0.865841977427
    >>> 2016-07-28 15:56:08.182477 Iteration 8000 overall accuracy 0.804960917114
    >>> 2016-07-28 15:56:08.182606 Iteration 8000 mean accuracy 0.723366231692
    >>> 2016-07-28 15:56:08.182786 Iteration 8000 mean IU 0.450797861465
    >>> 2016-07-28 15:56:08.182920 Iteration 8000 fwavacc 0.715707726933
    >>> 2016-07-28 16:08:59.654618 Begin seg tests
    >>> 2016-07-28 16:11:45.822145 Iteration 12000 loss 0.72744285975
    >>> 2016-07-28 16:11:45.822326 Iteration 12000 overall accuracy 0.829063382524
    >>> 2016-07-28 16:11:45.822399 Iteration 12000 mean accuracy 0.728383144052
    >>> 2016-07-28 16:11:45.822697 Iteration 12000 mean IU 0.470663378357
    >>> 2016-07-28 16:11:45.822794 Iteration 12000 fwavacc 0.738958147784
    >>> 2016-07-28 16:24:43.682142 Begin seg tests
    >>> 2016-07-28 16:27:32.582254 Iteration 16000 loss 0.671988376899
    >>> 2016-07-28 16:27:32.582414 Iteration 16000 overall accuracy 0.834907019939
    >>> 2016-07-28 16:27:32.582577 Iteration 16000 mean accuracy 0.730903583042
    >>> 2016-07-28 16:27:32.582713 Iteration 16000 mean IU 0.474480346083
    >>> 2016-07-28 16:27:32.582808 Iteration 16000 fwavacc 0.746916759749
可见，最终主要指标都出现大幅提升，说明训练有效！


**SegNet Training**  

根据调整方案1)，采用fixed的学习率1e-10，monentum为0.99。根据2），将softmaxloss层的normalize设置为true。  
同样，经过4000次迭代以后，我们发现修改后的参数模型，loss可以不断减小，即表示可以收敛，而且收敛速度快于Fcn32s网络，但是最主要的指标参数也依然没有出现变化，需等待进一步训练结果。  

    >>> 2016-07-27 16:36:01.449254 Iteration 4000 loss 3.04294351251  
    >>> 2016-07-27 16:36:01.449524 Iteration 4000 overall accuracy 0.733180213097  
    >>> 2016-07-27 16:36:01.449573 Iteration 4000 mean accuracy 0.047619047619  
    >>> 2016-07-27 16:36:01.449706 Iteration 4000 mean IU 0.0349133434808  
    >>> 2016-07-27 16:36:01.449798 Iteration 4000 fwavacc 0.537553224877  
由于之前的Fcn32s网络实现了有效训练，我们将相关调整也作用于SegNet网络上（选用相同的训练参数），测试结果。

正如预期，SegNet也可以较为快速的收敛，并且相应层参数均发生改变。最后，我们对整个网络进行训练测试，其结果如下：

    >>> 2016-07-28 16:47:44.193496 Iteration 4000 loss 1.34012441367
    >>> 2016-07-28 16:47:44.193629 Iteration 4000 overall accuracy 0.733180213097
    >>> 2016-07-28 16:47:44.193757 Iteration 4000 mean accuracy 0.047619047619
    >>> 2016-07-28 16:47:44.194040 Iteration 4000 mean IU 0.0349133434808
    >>> 2016-07-28 16:47:44.194139 Iteration 4000 fwavacc 0.537553224877
结果发现主要参数指标并没有改变，对相关层进行检查，发现decoder层(conv1_1_D)权重并未有效地学习。然而，对conv1_1_D的权重进行初始化，依然不能使得网络有效地收敛，如下：

    >>> 2016-07-28 19:19:44.126946 Iteration 24254 loss 1.30797043911
    >>> 2016-07-28 19:19:44.127122 Iteration 24254 overall accuracy 0.733161065479
    >>> 2016-07-28 19:19:44.127300 Iteration 24254 mean accuracy 0.0476180853631
    >>> 2016-07-28 19:19:44.127435 Iteration 24254 mean IU 0.0349127414014
    >>> 2016-07-28 19:19:44.127536 Iteration 24254 fwavacc 0.537539625039

接下来，调整网络模型，设置convolution层的参数为type='gaussian', std=0.01。我们发现依然很难收敛。于是，为了验证添加多个crop层是否正确，通过之前已经成功在CamVid训练集上测试的SegNet网络，再使用更改后的Caffe在新的网络上（添加多个crop层）进行测试。我们发现同样在CamVid上训练，SegNet(1e-3)似乎能有比Fcn32s(1e-4)更快的收敛速度。



**小结**

对于fcn32s网络来说，由于一开始并没有弄清楚各个训练参数和网络模型参数的意思，始终将vgg16对应层进入训练，相反最后的score_fr和upscore层并没有进行训练;除此以外，除vgg16对应层外的如fc6_conv等并没有有效初始化，无法实现参数更新。而且，当训练参数如base_lr设置太小（如1e-10）也会导致训练收敛速度过慢。当然，normalize softmax loss也是必须的。

PS:　对于fcn32s相较于SegNet来说，decoder部分层数少，因而训练过程速度要快于后者。
