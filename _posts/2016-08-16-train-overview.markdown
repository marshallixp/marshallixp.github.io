---
layout: post
title:  "Train Overview"
date:   2016-08-16 15:47:00
categories: segmentation
---

目前，我们已经基本上完成了SegNet和Fcn两种网络分别在CamVid和PascalVOC2012上的训练。
首先，SegNet较好地完成了CamVid训练，在没有进行样本扩增的条件下，取得了接近文献中所述的训练结果：
# Iteration 30000:  
Global acc = 0.85501 Class average acc = 0.5228 Mean Int over Union = 0.44517  

然而，该网络在PascalVOC2012训练集上，出现了一些问题。由于训练集中图片大小不一致，在后面的从pooling层max location坐标返回到进过upsample的数据时候会出现不对应的情况。因而，我们最先修改了网络中的upsample部分，用x2的方式扩大，再crop的方式，实现不同大小的图片数据训练。但是，无法进行多张图片的batch训练。不幸地是，我们发现网络非常难收敛，以至于总是落入局部，导致只能收敛到背景类。

接下来，我们重新修改了网络实现了以224x224大小的输入，并且从原图中分割出了左上、左下、右上、右下、中五个部分的自图片，构成新的数据集，再采用批量训练的方式进行训练。遗憾的是，SegNet无法获得很好的结果，可能是考虑到该网络更适合全尺寸图片作为输入的方式进行。

对于Fcn系列网络，我们依照文中所示方法，采用了分阶段的训练模式。先进行Fcn32s的训练，再Fcn16s，最后Fcn8s。根据这样的方法，我们在两个数据库中获得的结果如下：
*CamVid:*
Fcn32s # Iteration 100000:
Global acc = 0.80453 Class average acc = 0.47098 Mean Int over Union = 0.3897
Fcn16s # Iteration 100000:
Global acc = 0.81284 Class average acc = 0.4834 Mean Int over Union = 0.40864
Fcn8s 出现了网络倒退的情况，理论上较Fcn16s应没有太大的提升。

*PascalVOC2012:* (我们选取训练中最优结果)
Fcn32s
>>> 2016-08-15 10:59:04.305616 Begin seg tests
>>> 2016-08-15 11:03:10.741634 Iteration 24000 loss 0.527342150643
>>> 2016-08-15 11:03:10.741799 Iteration 24000 overall accuracy 0.842504675537
>>> 2016-08-15 11:03:10.741979 Iteration 24000 mean accuracy 0.740968773048
>>> 2016-08-15 11:03:10.742108 Iteration 24000 mean IU 0.497978761185
>>> 2016-08-15 11:03:10.742194 Iteration 24000 fwavacc 0.7562668472066
文中的结果IU为59.4

Fcn16s
>>> 2016-08-15 12:25:01.189435 Begin seg tests
>>> 2016-08-15 12:27:44.222110 Iteration 24000 loss 0.435023135992
>>> 2016-08-15 12:27:44.222216 Iteration 24000 overall accuracy 0.874859384552
>>> 2016-08-15 12:27:44.222333 Iteration 24000 mean accuracy 0.715943696788
>>> 2016-08-15 12:27:44.222608 Iteration 24000 mean IU 0.536879623865
>>> 2016-08-15 12:27:44.222697 Iteration 24000 fwavacc 0.796204639853
文中的结果IU为62.4

Fcn8s
结果目前还没有出来。文中结果IU为62.7

总结：
Fcn系列网络的缺点：
1) Fcn的Deconvolution层学习upsample它的输入特征图机制并没有保留Spatial correlation. 
2) 由于Fcn最多只调用到conv3层，没有用到精度更高的特征图，会导致失去边缘信息
3) 重用encoder部分的特征图，增大了内存的开销
4) 必须采用分阶段训练方法，也就是说如果最初的Fcn32s训练不好，最终的Fcn8s训练也不会有太大的提升。