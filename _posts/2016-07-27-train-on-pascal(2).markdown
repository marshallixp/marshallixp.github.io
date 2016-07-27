---
layout: post
title:  "Train on Pascal(2)"
date:   2016-07-27 15:09:00
categories: segmentation
---

由于前一次训练中发现网络并没有收敛，经过查阅资料，我们决定采用以下调整方案：
1) Reduce learning rate by a factor of 10
2) Normalize softmax loss (divide with the number of pixels -> normalize: true)
3) Try net surgery
4) mean RGB / BGR ?

Make sure you do not make the model to learn from the background (ignore_label=255) 

IU score / pixel-wise accuracy are more important



