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
