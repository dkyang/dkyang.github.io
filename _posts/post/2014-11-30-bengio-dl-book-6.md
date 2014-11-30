# Bengio Deep Learning 阅读笔记 —— Chapter 6 Feedforward Deep Networks
## 随便说两句
Yoshua Bengio大神最近把即将出版的deep learning新书放在了网上，去软件所参加的读书会也指定了这本书作为阅读材料。前段时间看了一点，现在还是赶紧看看赶上软件所的进度吧。

今天把第六章看了一下，发现写的确实挺不错，易懂而且提到了一些本质的东西，所以还是准备有空把阅读笔记都做一下。

## 6.1 Formalizing and Generalizing Neural Networks
神经网络与一般的机器学习模型相同，都包含了以下四个方面内容：

1. 定义带参数的函数族。 输出函数的参数从训练样本中训练得到。
2. 定义损失函数。 用于衡量模型的输出和target输出的差异。
3. 定义training criterion以及regularizer。 

$$E=MC^2$$