---
layout:     post
title:      Bengio Deep Learning 阅读笔记 —— Chapter 8 Optimization for training deep models
category:   post
description: Bengio深度学习第8章阅读笔记
tags: 深度学习 阅读笔记
---
## 8.1 Optimization for model training
### 8.1.3 Cliffs and Exploding Gradients
目标函数的二阶结构会导致鞍点和病态问题的出现，这对优化算法带来了挑战。常用的二阶方法如momentum通过动态调节步长大小解决这个问题，即在高曲率方向上减小步长，在低曲率方向上增加步长。但这样的方法在训练deep model时不太适用，因为deep model引入了更多的非线性，其目标函数不具有近似对称的结构，而是会有突然的增大或下降，类似于cliff。高度的非线性会导致过大的导数，从而带来参数更新的反弹，前面多轮迭代的结果可能会因cliff而消弥，如下图所示：

![Cliff](/images/bengio/cliff.jpg)

因此对于deep model可以直接对更新步长进行截断，以解决cliff带来的梯度过大问题。


这部分为什么momentum适用于解决目前函数具有对称性的场景还不是很明白。

## 8.1.4 Vanishing and Exploding Gradients
基于链式法则和神经网络逐层依赖的结构，我们知道神经网络导数的计算是一个相乘的关系，即：
	
	f' = f1' * f2' * f3' * .... * ft-1' * ft'

这实际上是一种非线性叠加。在非线性叠加的导数中，可能有部分过大或过小的值。这些过小的值之间相乘会导致最终结果很小，反之过大的数相乘导致结果很大，这样神经网络中的梯度很容易vanish或explod，带来了神经网络训练的难度。

![Non-linear](/images/bengio/non-linear.jpg)

特别的，RNN这种递归结构导致学习long-term dependency变得很困难。梯度可以用下式分解：

![deriv1](/images/bengio/formular1.jpg)

其中dsT/dst为：

![deriv2](/images/bengio/formular2.jpg)

可见梯度可以看做dsT/dst的加权平均，而dsT/dst会随着式中各因子的项数而指数增长，变得过大过小。因此要学习long-term dependency也会遇到gradients vanish/explode的问题，RNN的训练也变得更加困难。

## 8.2.3 Coordinate descent
简单介绍了下坐标下降，即每次选取一个坐标进行优化。更通用的是block coordinate descent，每次选取的是坐标的子集。这种方法的思想是坐标可以分为组，而仅用相关组进行优化可能比用整体的变量更加高效。

