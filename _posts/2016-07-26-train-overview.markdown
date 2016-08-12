---
layout: post
title:  "Train Overview"
date:   2016-07-26 16:44:59
categories: segmentation
---
**solver.prototxt参数说明**
train_net/test_net ： 网络定义文件的路径
test_iter : 如果数据库有1000个样本，batch_size为10，则需要100次iterations. 因而定义了test_iter，则表示测试阶段每次数据量batch_size
test_interval : 训练阶段，定义迭代的次数进行测试，如500次迭代一次测试
base_lr ： 初始学习速率
- fixed : 保持base_lr不变
- step ： 需设置stepsize, 返回base_lr * gamma ^ (floor(iter / stepsize))
- inv : 需设置power，返回base_lr * (1 + gamma*iter)^(-power)
- poly : 进行多项式误差，返回base_lr*(1-iter/max_iter)^power
- sigmoid ： 进行sigmoid衰减，返回base_lr*(1/(1+exp(-gamma*(iter-stepsize))))
momentum : 含该参数的sgd
weight_decay : 权重衰减
lr_policy : 学习参数的衰减方式
gamma, power
iter_size : 实际的batch_size等于iter_size x 网络定义的batch_size，因此每一次迭代的loss是iter_size次迭代的总和，再除以iter_size。如此设置是应对直接使用大batch_size会导致out of memory，借助这个方法可以使用较小的batch_size来完成实际上处理相同数据量的工作。

**Training**  
solver = caffe.SGDSolver("solver.prototxt")  
如果进行fine-tune，则需要调用  
solver.net.copy_from('xxx.caffemodel')
可以继续未完成的训练，通过调用  
solver.restore('xxx.solverstate')  

在fine-tune过程中，参数设置如下实现训练过程对模型权重的更改。
param {  
    lr_mult: 1  
    decay_mult: 1  
}  
而param{lr_mult:0}表示为保留原有模型权重（载入模型）。  

example/net_surgery.ipynb是有效地介绍如何使用surgery的内容。  




**训练过程：**
1）样本预处理，均值化
2）选择网络，ReLU
3）初始化权重（xavier）
4）babysitting the learning process
5）由cross到finer处理

**接下来,：**
1）参数更新机制（adam，agagrad,）
2）学习速率调节
3）Dropout
4）梯度检查
5）模型聚合

<font color=#FF0000 size=4>
**Patchwise training is loss sampling**  
Sampling in patchwise training can correct class imbalance, and mitigate the spatical correlation of dense patches. In fully convolutional training, class balance can also be achieved by weighting the loss, and loss sampling can be used to address spatial correlation.
**Class Balancing**  
Fully convolutional training can balance classes by weighting or sampling the loss. No need to balance class.
</font>