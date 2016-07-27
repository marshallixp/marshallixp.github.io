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

要点：
a) Make sure you do not make the model to learn from the background (ignore_label=255)  
b) IU score / pixel-wise accuracy are more important
c) Check each layer to see it's initialized with correct values


**SegNet Training**
根据调整方案1)，采用fixed的学习率1e-10，monentum为0.99。根据2），将softmaxloss层的normalize设置为true。  
同样，经过4000次迭代以后，我们发现修改后的参数模型，loss可以不断减小，即表示可以收敛，而且收敛速度快于Fcn32s网络，但是最主要的指标参数也依然没有出现变化，需等待进一步训练结果。  

    >>> 2016-07-27 16:36:01.449254 Iteration 4000 loss 3.04294351251  
    >>> 2016-07-27 16:36:01.449524 Iteration 4000 overall accuracy 0.733180213097  
    >>> 2016-07-27 16:36:01.449573 Iteration 4000 mean accuracy 0.047619047619  
    >>> 2016-07-27 16:36:01.449706 Iteration 4000 mean IU 0.0349133434808  
    >>> 2016-07-27 16:36:01.449798 Iteration 4000 fwavacc 0.537553224877  



**Fcn32s Training**
根据调整方案1），目前采用fixed的学习率，取值1e-9。根据2），将softmaxloss层的normalize设置为true。  
根据修改后的参数模型，经过4000次的迭代以后，我们发现loss是在不断减小，说明模型可以收敛，但是主要的指标参数依然没有改变，需要等待进一步的结果。  

    >>> 2016-07-27 15:58:21.062000 Iteration 4000 loss 3.04448500594  
    >>> 2016-07-27 15:58:21.062184 Iteration 4000 overall accuracy 0.733180213097  
    >>> 2016-07-27 15:58:21.062357 Iteration 4000 mean accuracy 0.047619047619  
    >>> 2016-07-27 15:58:21.062504 Iteration 4000 mean IU 0.0349133434808  
    >>> 2016-07-27 15:58:21.062591 Iteration 4000 fwavacc 0.537553224877  

