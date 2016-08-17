---
layout: post
title:  "Train on Pascal(4)"
date:   2016-08-12 14:20:00
categories: segmentation
---

根据过去近两周的工作，又仔细地研读了相关论文，针对PascalVOC2012数据集，采用了依照原论文的分阶段训练方式进行。首先，重新矫正了Fcn32s的网络参数和solver.prototxt，设置为20的iter_size，即用大小为20的batch_size进行网络参数更新。拟采用先进行Fcn32s的训练（Fcn-vgg16），然后进行Fcn16s(导入Fcn32s训练结果)训练，最后将Fcn16s训练结果导入Fcn8s完成三步分阶段训练。

关于L.Python层的batch_size修改问题，不仅可以从前文所说的进行修改，也可以在module对于对应的py文件中添加slef.batch_size= param['batch_size']进行更改，同时在forward函数中进行top的相应修改。


**Pascal VOC 2012 训练步骤：**
Fcn32s -> Fcn16s -> Fcn8s 或 Fcn32s -> Fcn8s

Step1. 首先对Fcn32s进行训练，主要参数如下：
base_lr: 1e-4
momentum: 0.9
iter_size: 20
max_iter: 100000
weight_decay: 0.0005

较优训练结果：
>>> 2016-08-15 10:59:04.305616 Begin seg tests
>>> 2016-08-15 11:03:10.741634 Iteration 24000 loss 0.527342150643
>>> 2016-08-15 11:03:10.741799 Iteration 24000 overall accuracy 0.842504675537
>>> 2016-08-15 11:03:10.741979 Iteration 24000 mean accuracy 0.740968773048
>>> 2016-08-15 11:03:10.742108 Iteration 24000 mean IU 0.497978761185
>>> 2016-08-15 11:03:10.742194 Iteration 24000 fwavacc 0.7562668472066

Step2. 训练Fcn16s网络，将Fcn32s网络训练结果导入，主要参数如下：
base_lr: 1e-6
momentum: 0.9
iter_size: 1
max_iter: 100000
weight_decay: 0.0005
较优训练结果：
>>> 2016-08-15 12:25:01.189435 Begin seg tests
>>> 2016-08-15 12:27:44.222110 Iteration 24000 loss 0.435023135992
>>> 2016-08-15 12:27:44.222216 Iteration 24000 overall accuracy 0.874859384552
>>> 2016-08-15 12:27:44.222333 Iteration 24000 mean accuracy 0.715943696788
>>> 2016-08-15 12:27:44.222608 Iteration 24000 mean IU 0.536879623865
>>> 2016-08-15 12:27:44.222697 Iteration 24000 fwavacc 0.796204639853
可以看到，从统计数据上来说较前一模型有4%左右的提升。

Step3. 训练Fcn8s网络，将Fcn16s的训练结果导入，
base_lr: 1e-8
momentum: 0.99
iter_size: 20
max_iter: 100000
weight_decay: 0.0005
较优训练结果：

 
