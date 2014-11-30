---
layout:     post
title:      Network in Network 文献阅读
category:   post
description: 利用MLP代替线性卷积核，近似非线性函数
tags: 文献阅读 深度学习 CNN 计算机视觉
---
# Network in Network文献阅读
## 简介
Network in Network的得名，来源于本文的网络结构在传统CNN中又使用了multilayer perceptron。具体来说，就是用MLP替换了传统CNN中的线性卷积核，通过这样的方式可以学习得到非线性的判别函数，从而得到更好地分类性能。

CNN中的卷积核对数据块来说实际上是一种广义线性模型（GLM），GLM假定了样本的潜在概念是线性可分的，这是一种较低层级的抽象，对相同概念的变体不变性较差。在实际中，潜在概念往往是高度非线性的，因此我们可以用较复杂的结构替换线性卷积核以改进CNN。二者的结构区别如下图：

![Network-Arch](/images/nin/NIN-structure.jpg)

## Maxout Network与NIN
NIN延续了maxout network的思想，都是对CNN的GLM函数做改进。不同的是，maxout network对affine feature maps做max pooling（affine feature maps就是线性卷积的直接输出而不施加激励函数），通过对一系列线性函数求最大值可以得到piecewise linear approximator从而近似任意凸函数，因此maxout network通过近似局部patch的凸函数可以对样本在凸集中概念构建超平面。maxout network的缺点是添加了先验，要求样本的潜在概念在输入的凸集中，这种先验不是一定能满足。NIN结构用MLP这种universal function approximator代替convex function approximator，可以对不同分布的潜在概念建模。

## 全局平均池化（global average pooling）
传统CNN中的全连接层容易过拟合，文章提出用global average pooling代替全连接层，直接对每个feature map取平均作为softmax层的输入。这种策略有如下优点：
1. 增强了feature map与类别之间的对应关系，更具解释性；
2. global average pooling没有参数需要优化，减少了参数个数，更不易过拟合，也起到了正则的作用；
3. 对空间信息进行了求和，对输入的空间变换更鲁棒；
## 总结
全文的创新主要有两点，一是利用多层感知机对输入卷积，可以更好地建模局部patch；二是用global average pooling层替换了全连接层，作为结构正则减少了全局过拟合。通过这两点，NIN在多个数据集上达到了state-of-art的效果。