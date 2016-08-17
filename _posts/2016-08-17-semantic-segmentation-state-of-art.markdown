---
layout: post
title:  "Semantic Segmentation最新进展"
date:   2016-08-17 10:17:00
categories: segmentation
---

# Semantic Segmentation基本原理
Semantic Segmentation (简称SS)是针对图像进行像素级的分类，即将每一个图像中的像素进行有效识别，从而实现对场景中不同目标，特别是不同目标之间的边缘进行有效识别的方法。目前，在SS处理的具体实现方法大都通过深度学习中卷积神经网络作为encoder层，将原始图片逐步降维，生成一系列不同尺寸的特征图（feature map）。再通过以上采样（upsample）作为decoder，涉及多个不同名词：deconvolution、transposed convolution等，实现从小尺度还原到原始图片尺度的像素级预测。由于在encoder部分，往往采用VGG16网络结构训练出的参数模型进行fine-tuning，那么主要工作就是如何实现将特征图还原到原始尺寸。根据这个思想，文献[1]较早提出来Fully Convolutional层（Fcn32s/16s/8s）替代VGG16的全连接（Fully Connected）层，再通过反向卷积层的双线性差值进行上采样的初始化，并且加以参数学习能力，实现从特征图还原到原始尺寸的预测。然而Fcn系列网络的训练采用分阶段进行，而且最多只调用到encoder第三层的特征图，缺少更高精度的像素信息，同时具有较大的内存开销。但是其思想具有很高的实用价值，end-to-end训练的简易性，无须定义patch进行扫描，并且性能上取得了突破性的成果。后来的研究者大都基于该思想设计网络，包括Noh等人[2][3]提出的运用encoder特征图的max location替代直接使用特征图，进行非线性上采样方法，Vijay等人[4]提出的encoder-decoder对称结构，Liu等人[5]运用特征组合的方式将局部信息和全局信息融合，Adam等人[6]运用留数网络作为encoder结合小规模的decoder等工作大都能看到Fcn思想的影子，只是结构上多有不同。

但是，从目前的研究进展看，依然还存在着网络泛化能力不足，精度不高（IU<80%）等问题，特别是在复杂场景中出现的小目标识别、边缘信息识别等障碍。我们接下来看看近期该领域的研究进展。



# SS最新进展（深度学习）



[1] Long, Jonathan, Evan Shelhamer, and Trevor Darrell. "Fully convolutional networks for semantic segmentation." Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2015. 
[2] Noh, Hyeonwoo, Seunghoon Hong, and Bohyung Han. "Learning deconvolution network for semantic segmentation." Proceedings of the IEEE International Conference on Computer Vision. 2015.
[3] Hong, Seunghoon, Hyeonwoo Noh, and Bohyung Han. "Decoupled deep neural network for semi-supervised semantic segmentation." Advances in Neural Information Processing Systems. 2015.
[4] Vijay Badrinarayanan, Ankur Handa and Roberto Cipolla "SegNet: A Deep Convolutional Encoder-Decoder Architecture for Robust Semantic Pixel-Wise Labelling". arXiv:1505.07293   
[5] Liu, Wei, Andrew Rabinovich, and Alexander C. Berg. "Parsenet: Looking wider to see better." arXiv preprint arXiv:1506.04579 (2015).
[6] Adam Paszke, Abhishek Chaurasia, Sangpil Kim, Eugenio Culurciello "ENet: A Deep Neural Network Architecture for Real-Time Semantic Segmentation". arXiv:1606.02147.