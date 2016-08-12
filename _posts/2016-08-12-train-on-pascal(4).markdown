---
layout: post
title:  "Train on Pascal(4)"
date:   2016-08-12 14:20:00
categories: segmentation
---

根据过去近两周的工作，又仔细地研读了相关论文，针对PascalVOC2012数据集，采用了依照原论文的分阶段训练方式进行。首先，重新矫正了Fcn32s的网络参数和solver.prototxt，设置为20的iter_size，即用大小为20的batch_size进行网络参数更新。拟采用先进行Fcn32s的训练（Fcn-vgg16），然后进行Fcn16s(导入Fcn32s训练结果)训练，最后将Fcn16s训练结果导入Fcn8s完成三步分阶段训练。