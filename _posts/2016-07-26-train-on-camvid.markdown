---
layout: post
title:  "Train on CamVid"
date:   2016-07-26 17:09:59
categories: segmentation
---
原有camvid拥有33个类：

64 128 64   Animal
192 0 128   Archway
0 128 192   Bicyclist
0 128 64    Bridge
128 0 0     Building
64 0 128    Car
64 0 192    CartLuggagePram
192 128 64  Child
192 192 128 Column_Pole
64 64 128   Fence
128 0 192   LaneMkgsDriv
192 0 64    LaneMkgsNonDriv
128 128 64  Misc_Text
192 0 192   MotorcycleScooter
128 64 64   OtherMoving
64 192 128  ParkingBlock
64 64 0     Pedestrian
128 64 128  Road
128 128 192 RoadShoulder
0 0 192     Sidewalk
192 128 128 SignSymbol
128 128 128 Sky
64 128 192  SUVPickupTruck
0 0 64      TrafficCone
0 64 64     TrafficLight
192 64 128  Train
128 128 0   Tree
192 128 192 Truck_Bus
64 0 64     Tunnel
192 192 0   VegetationMisc
0 0 0       Void
64 192 0    Wall

**SegNet postprocess**
python /Segnet/Scripts/compute_bn_statistics.py /SegNet/Models/segnet_train.prototxt /SegNet/Models/Training/segnet_iter_10000.caffemodel /Segnet/Models/Inference/




**SegNet + Pretrained VGG16**  
# Iteration 10000:  
Global acc = 0.85101 Class average acc = 0.46322 Mean Int over Union = 0.40203 
# Iteration 20000:  
Global acc = 0.85167 Class average acc = 0.52274 Mean Int over Union = 0.44069 
# Iteration 30000:  
Global acc = 0.85501 Class average acc = 0.5228 Mean Int over Union = 0.44517  

**SegNet modified (修正crop层，可以自动实现不同层之间的匹配)**
# Iteration 26000:
Global acc = 0.81241 Class average acc = 0.47571 Mean Int over Union = 0.3939
低于原有模型训练结果，或许跟参数设置有关系。


**Fcn32s + Pretrained VGG16**
# Iteration 56000:
Global acc = 0.79462 Class average acc = 0.4331 Mean Int over Union = 0.35841
# Iteration 100000:
Global acc = 0.80453 Class average acc = 0.47098 Mean Int over Union = 0.38977

**Fcn16s + Pretrained Fcn32s**
# Iteration 100000:
Global acc = 0.81284 Class average acc = 0.4834 Mean Int over Union = 0.40864

**Fcn8s + Pretrained Fcn16s**
网络出现退化情况

**Fcn8s + Pretrained VGG16**
# Iteration 56000:
Global acc = 0.70828 Class average acc = 0.36595 Mean Int over Union = 0.28398


**总结：**
在针对道路场景的语义分割训练中，我们主要参考了Fcn[1], SegNet[2], ENet[3]，这三篇文章。其中，Fcn是最早提出来通过双线性差值upsampling的方式实现end-to-end训练，生成与原图相同尺寸的语义分割map，并采用了fine-tune方法通过加载VGG16预先训练好的权重进行局部训练。Fcn针对不同尺度的细节表现和分割平滑，从encoder中的不同层提取相应的map进行组合的方式，实现综合finer和coarser部分。SegNet提出了一种对称结构的encoder-decoder网络，encoder部分省略了Vgg16的Full connected层。而在实际训练中，我们发现相较于前者，SegNet更难收敛，但是对细节的描述更加精准。而ENet主要的工作是采用留数网络ResNet的encoder结构，能够实现更快速的inference过程（训练较慢）。对于decoder部分，认为主要就是实现对原图尺寸的还原，平滑分割边界，因而采用了较小的网络结构（可能是Fcn32s的decoder部分）。最后，我们还参考了comma.ai上提出的方法，由于decoder的作用是尺度还原和平滑边界，可以采用较小的网络结构，他们使用的是一个16x5x5核，步长2的upscaling方法。

[1] Liang-Chieh Chen, George Papandreou, Iasonas Kokkinos, Kevin Murphy, Alan L. Yuille "Semantic Image Segmentation with Deep Convolutional Nets and Fully Connected CRFs". arXiv:1412.7062  
[2] Vijay Badrinarayanan, Ankur Handa and Roberto Cipolla "SegNet: A Deep Convolutional Encoder-Decoder Architecture for Robust Semantic Pixel-Wise Labelling". arXiv:1505.07293   
[3] Adam Paszke, Abhishek Chaurasia, Sangpil Kim, Eugenio Culurciello "ENet: A Deep Neural Network Architecture for Real-Time Semantic Segmentation". arXiv:1606.02147.