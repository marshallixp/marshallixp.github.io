---
layout: post
title:  "Train on CamVid"
date:   2016-07-26 17:09:59
categories: segmentation
---

**camvid training and test**
python /Segnet/Scripts/compute_bn_statistics.py /SegNet/Models/segnet_train.prototxt /SegNet/Models/Training/segnet_iter_10000.caffemodel /Segnet/Models/Inference/




**SegNet + Pretrained VGG16**  

# Iteration 30000:  
Global acc = 0.85501 Class average acc = 0.5228 Mean Int over Union = 0.44517  
# Iteration 20000:  
Global acc = 0.85167 Class average acc = 0.52274 Mean Int over Union = 0.44069 
# Iteration 10000:  
Global acc = 0.85101 Class average acc = 0.46322 Mean Int over Union = 0.40203 

SegNet modified (修正crop层，可以自动实现不同层之间的匹配) 
# Iteration 16468:
Global acc = 0.81453 Class average acc = 0.46955 Mean Int over Union = 0.3876
# Iteration 26000:
Global acc = 0.81241 Class average acc = 0.47571 Mean Int over Union = 0.3939
低于原有模型训练结果，或许跟参数设置有关系。te


**Fcn32s + Pretrained VGG16**

# Iteration 56000:
Global acc = 0.79462 Class average acc = 0.4331 Mean Int over Union = 0.35841


