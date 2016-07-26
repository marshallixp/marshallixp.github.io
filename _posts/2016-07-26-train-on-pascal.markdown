---
layout: post
title:  "Train on Pascal"
date:   2016-07-26 17:09:59
categories: segmentation
---

SegNet Training

**Step1. 不训练（没有反向过程），直接测试。**
1）不用pretrained模型
2）使用pretrained模型
其中，1）结果如下：
	>>> 2016-07-25 15:29:18.270980 Iteration 0 loss 87.3365001902
	>>> 2016-07-25 15:29:18.271152 Iteration 0 overall accuracy 0.733180213097
	>>> 2016-07-25 15:29:18.271215 Iteration 0 mean accuracy 0.047619047619
	>>> 2016-07-25 15:29:18.271508 Iteration 0 mean IU 0.0349133434808
	>>> 2016-07-25 15:29:18.271601 Iteration 0 fwavacc 0.537553224877
相应，2）结果如下：
	>>> 2016-07-25 15:35:54.661197 Iteration 0 loss 87.3365001902
	>>> 2016-07-25 15:35:54.661321 Iteration 0 overall accuracy 0.733180213097
	>>> 2016-07-25 15:35:54.661488 Iteration 0 mean accuracy 0.047619047619
	>>> 2016-07-25 15:35:54.661621 Iteration 0 mean IU 0.0349133434808
	>>> 2016-07-25 15:35:54.661709 Iteration 0 fwavacc 0.537553224877
很明显，无论是否采用vgg16对没有经过训练网络的结果没有影响。
    
**Step2. 训练，再测试。**
solver.protoxt中参数根据文章中的设置如下：
train_net: "train.prototxt"
test_net: "val.prototxt"
test_initialization: false
test_iter: 736
test_interval: 999999999
display: 20
average_loss: 20
lr_policy: "fixed"
base_lr: 1e-4
momentum: 0.9
iter_size: 1 
max_iter: 100000
weight_decay: 0.0005
snapshot: 4000
snapshot_prefix: "snapshot/segnet"
solver_mode: GPU
    
1）使用pretrained VGG16模型，每次训练步长为4000，训练结果如下：
	>>> 2016-07-26 12:23:04.827459 Iteration 4000 loss 1.31175746871
	>>> 2016-07-26 12:23:04.827577 Iteration 4000 overall accuracy 0.733180213097
	>>> 2016-07-26 12:23:04.827639 Iteration 4000 mean accuracy 0.047619047619
	>>> 2016-07-26 12:23:04.827855 Iteration 4000 mean IU 0.0349133434808
	>>> 2016-07-26 12:23:04.828067 Iteration 4000 fwavacc 0.537553224877
在经过4000次迭代，大约2.5倍样本集后，发现loss在缓慢下降，但是其他测量结果没有改变。

	>>> 2016-07-26 17:29:48.209541 Iteration 48000 loss 1.56178289191
	>>> 2016-07-26 17:29:48.209808 Iteration 48000 overall accuracy 0.733170522908
	>>> 2016-07-26 17:29:48.209855 Iteration 48000 mean accuracy 0.0476187852268
	>>> 2016-07-26 17:29:48.209980 Iteration 48000 mean IU 0.0349132915075
	>>> 2016-07-26 17:29:48.210064 Iteration 48000 fwavacc 0.537546775601
    在经过48000次迭代以后，loss并没有收敛，训练失败，待续。


	Fcn32s Training
	首先，我们下载了训练好的模型fcn32s-heavy-model，发现经过finetune以后，结果会更好。
	Fcn32s fine-tune with pretrained heavy model 
	>>> 2016-07-11 17:09:09.316152 Begin seg tests
	>>> 2016-07-11 17:11:40.982484 Iteration 4000 loss 40536.5665925
	>>> 2016-07-11 17:11:40.982576 Iteration 4000 overall accuracy 0.924534222757
	>>> 2016-07-11 17:11:40.982608 Iteration 4000 mean accuracy 0.789070139032
	>>> 2016-07-11 17:11:40.982729 Iteration 4000 mean IU 0.68489801572
	>>> 2016-07-11 17:11:40.982804 Iteration 4000 fwavacc 0.867180920774

	>>> 2016-07-11 17:32:13.296804 Begin seg tests
	>>> 2016-07-11 17:34:39.726955 Iteration 8000 loss 40918.7736482
	>>> 2016-07-11 17:34:39.727035 Iteration 8000 overall accuracy 0.928679202249
	>>> 2016-07-11 17:34:39.727064 Iteration 8000 mean accuracy 0.812085149284
	>>> 2016-07-11 17:34:39.727181 Iteration 8000 mean IU 0.698756970877
	>>> 2016-07-11 17:34:39.727255 Iteration 8000 fwavacc 0.873438191274

	Fcn32s + pretrained heavy model without training
	>>> 2016-07-11 17:38:19.174321 Begin seg tests
	>>> 2016-07-11 17:40:52.292152 Iteration 0 loss 45737.5351437
	>>> 2016-07-11 17:40:52.292247 Iteration 0 overall accuracy 0.926090782406
	>>> 2016-07-11 17:40:52.292278 Iteration 0 mean accuracy 0.817075271365
	>>> 2016-07-11 17:40:52.292399 Iteration 0 mean IU 0.695001155057
	>>> 2016-07-11 17:40:52.292475 Iteration 0 fwavacc 0.869301769793

	其次，我们利用pretrained VGG16进行训练，目前得到的结果显示，loss会降低，但是其他测量结果并没有发生改变，如下：
	>>> 2016-07-26 12:48:33.904036 Iteration 8000 loss 3916726.69319
	>>> 2016-07-26 12:48:33.904141 Iteration 8000 overall accuracy 0.733180213097
	>>> 2016-07-26 12:48:33.904256 Iteration 8000 mean accuracy 0.047619047619
	>>> 2016-07-26 12:48:33.904529 Iteration 8000 mean IU 0.0349133434808
	>>> 2016-07-26 12:48:33.904619 Iteration 8000 fwavacc 0.53755322487

	>>> 2016-07-26 13:39:00.477857 Iteration 16000 loss 3646240.50901
	>>> 2016-07-26 13:39:00.477998 Iteration 16000 overall accuracy 0.733180213097
	>>> 2016-07-26 13:39:00.478118 Iteration 16000 mean accuracy 0.047619047619
	>>> 2016-07-26 13:39:00.478241 Iteration 16000 mean IU 0.0349133434808
	>>> 2016-07-26 13:39:00.478323 Iteration 16000 fwavacc 0.537553224877

	>>> 2016-07-26 17:03:43.480082 Iteration 48000 loss 3076769.60242
	>>> 2016-07-26 17:03:43.480186 Iteration 48000 overall accuracy 0.733180213097
	>>> 2016-07-26 17:03:43.480288 Iteration 48000 mean accuracy 0.047619047619
	>>> 2016-07-26 17:03:43.480508 Iteration 48000 mean IU 0.0349133434808
	>>> 2016-07-26 17:03:43.480592 Iteration 48000 fwavacc 0.537553224877
	经过近50000次的训练，发现其中loss并没有收敛，而且另外4个指标也完全没有任何改变，和未经过训练的结果相同。可见，这里存在什么问题还没有弄清楚，待续。
	

