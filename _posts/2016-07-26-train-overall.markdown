---
layout: post
title:  "solver文件各参数说明文档"
date:   2016-07-26 16:44:59
categories: segmentation
---
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