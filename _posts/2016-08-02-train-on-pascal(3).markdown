---
layout: post
title:  "Train on Pascal(3)"
date:   2016-08-02 11:31:00
categories: segmentation
---

通过测试，验证了通过voc_layer均值化后的data和转换成1x1xHxW的label数据的正确。label是包含0~20以及255的同尺寸数据。label:0对应background，而1~20对应不同目标类别。现在的问题是，运用segnet网络训练后，只能收敛到背景分类。

目前，已经完成的训练及测试包括：
a. Fcn32s on Pascal
b. Fcn32s on CamVid
c. SegNet on CamVid
以上都有接近文章的效果ne

而SegNet on Pascal依然无法实现，正在测试几种改进方案：
1）增加Dropout
2）利用basic segnet进行测试
3）生成符合DenseImageDataLayer的label数据集
4）只采用少量（～10）样本进行训练

方案1）增加了两种不同的dropout层，全覆盖和半覆盖，包括结合不同的学习速率进行训练，依然未能解决问题。

方案2）basic segnet似乎也未见效，依然落入局部最小值，输出全为背景标记。

方案3）通过在网络测试过程中，获取的经过处理后的label图片，保存至SegmentationClass-SegNet，再调用DenseImageDataLayer进行读取。经测试，依然无效。

方案4）采用10个训练样本，能看到loss是在不断减小，且经过大约5000次迭代(1e-4, 0.9)以后，不全是输出到背景。且通过更改weight_decay=0.5，能收敛到非背景目标。

至此，我们采取的大多数测试并不能使网络有效收敛，于是，我们继续采用原文中的处理方法，将图片crop成固定大小224x224，并且通过采样不同区域，进行augmentation。再采用固定每层尺寸大小的网络，即不使用crop层，进行测试。

改回upsample???




