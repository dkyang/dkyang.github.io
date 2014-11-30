---
layout:     post
title:      Early Hierarchical Contexts Learned by Convolutional Networks for Image Segmentation 文献阅读
category:   post
description: 百度慧眼识人大赛获胜算法，用CNN结合局部信息做图像分割
tags: 文献阅读 深度学习 CNN 计算机视觉 图像分割
---
# Early Hierarchical Contexts Learned by Convolutional Networks for Image Segmentation文献阅读
这篇文章的算法在百度去年慧眼识人大赛上获得了第一名，用CNN做了以人体为前景的图像分割。
这种分割更像是一种逐像素的分类，每个像素用其局部图像块的信息表示，再结合了现在非常火热的CNN。算法效果还是非常好的，超过了比赛第二名8%（显著性检测+迭代KNN和grabcut）的精度，看完只有一句话：CNN大法好啊！

文章的思路并不复杂，就是将一个像素用层次化的上下文（hierarchical context）表示，直接作为CNN的输入，最终的输出包含两个unit，表示这个像素是否为前景。具体来说，一个像素的层次上下文由3个56 * 56的局部图像块表示，这三个图像块表示了同一个像素不同尺度的局部。首先从224 * 224的图像块中截取中心56 * 56的区域得到最小的上下文，再将224 * 224图像块下采样到112 * 112，同样截取56*56区域，最后将224 * 224图像块下采样到56 * 56，得到最大的上下文。这三个56 * 56的图像块分别输入3个5-stage CNN列，再通过一个三层感知机完成训练。

![Architecture](/images/cnn-segmentation/cnn-segmentation.jpg)

虽然精度够高，但本文离实际应用还是有些距离的。主要是test的耗时太久，在Titan GPU上的耗时都超过了1分钟。另外本文的思路很容易就可以迁移到肤色检测上，常用的肤色检测算法都只考虑单个像素，如果用本文这种结合上下文的方式效果一定不差，有空可以做下这方面的实验。