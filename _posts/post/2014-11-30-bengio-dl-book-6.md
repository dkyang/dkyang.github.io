# Bengio Deep Learning 阅读笔记 —— Chapter 6 Feedforward Deep Networks
## 随便说两句
Yoshua Bengio大神最近把即将出版的deep learning新书放在了网上，去软件所参加的读书会也指定了这本书作为阅读材料。前段时间看了一点，现在还是赶紧看看赶上软件所的进度吧。

今天把第六章看了一下，发现这本书写的确实挺不错，易懂而且提到了一些本质的东西，所以还是准备有空把阅读笔记都做一下。

## 6.1 Formalizing and Generalizing Neural Networks
神经网络与一般的机器学习模型相同，都包含了以下四个方面内容：

1. 定义带参数的函数族。 输出函数的参数从训练样本中训练得到。
2. 定义损失函数。 用于衡量模型的输出和target输出的差异。
3. 定义training criterion以及regularizer。 训练的目标是最小化loss的期望，但数据的分布P(X,Y)未知，因此一般都采用近似的手段，比如最小化训练集loss的经验平均。
4. 优化步骤。 近似优化training criterion。


## 6.2 Parametrizing a Learned Preditor
本节展开讨论了上面四部分的内容。

### 6.2.1 Family of Functions
多层神经网络定义的函数用于组合仿射变换和非线性，并且可以用来近似平滑函数，hidden unit的个数越多近似越精确，从这点也可以理解较大模型对神经网络的意义。

神经网络中用于添加非线性的函数有好几种，常用的包括ReLU、maxout、sigmoid、softmax等。

### 6.2.2 Loss Function and Conditional Log-Likelihood
这里从统一的视角来看loss function。loss function可以被定义为条件log似然度，不同的loss function假定了(Y,X)有不同的分布。比如squared error可以由对均值为f\_theta(x)的高斯分布取log条件似然得到，交叉熵error可以由概率为f\_theta(x)的伯努利分布得到。

总体来说，带参数的概率P(Y|w)都可以由P(Y|X)构建，其中w由f\_theta(x)表示，这样negative log-似然度L(X,Y) = -logP(Y|w=f\_theta(X))。

### 6.2.3 Training Criterion and Regularizer
前面提到training criterion的目标是最小化E_X,Y[-log P_theta(Y|X)]，但P(X,Y)未知，可以用training loss的经验平均代替，即1/n * [sum(L(f_theta(x), y))]。

常用的正则包括L1、L2，以及结合L1和L2的elastic net。L1对参数有着常数梯度，因此相对L2更可能令参数为0。另一方面，L1对较大的参数有更多地容忍，L2对大参数由较多的惩罚。

### 6.2.4 优化步骤
这部分介绍了一些优化方法，没有什么特别值得记录的。

## 6.3 Flow Graphs and 反向传播
这部分对反向传播的介绍还是挺不错的。反向传播的基本思想就是结合偏导数的链式法则和神经网络参数层与层之间的直接依赖性，用类似动态规划的方式，从输出向输入逐层计算偏导数。层与层之间变量的依赖可以用flow graph来表示，（待解释流图和反向传播的关系）

## 6.4 
线性模型无法表示复杂函数。

深度也是**先验**。

## 6.5 
kernel和特征工程都可视为一种先验。



