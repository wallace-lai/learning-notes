# 【浙大机器学习】第一章

作者：wallace-lai <br/>
发布：2023 <br/>
更新：2023 <br/>

## P1 机器学习定义

Arthur Samuel对机器学习的定义：

> Machine Learning is Fields of study that gives computers the ablity to learn without being explicityly programmed

> 机器学习是这样的领域，它赋予计算机学习的能力，这种学习能力不是通过显著式编程获得的

Tom Mishell对机器学习的定义

> A computer program is said to learn from experience E with respect to some task T and some performance measure P, if its performance on T T, as measured by P, improves with experience E

> 一个计算机程序被称为可以学习，是指它能够针对某个任务T和某个性能指标P，从经验E中学习。这种学习的特点是，它在T上的被P所衡量的性能，会随着敬仰E的增加而提高

## P2 机器学习的分类

按照任务是否需要和环境交互获得经验，机器学习可分为：
- 监督学习
- 强化学习

对监督学习进行更细致分类：

（1）按照训练数据是否存在标签，将监督学习分为：
- 传统监督学习
- 非监督学习
- 半监督学习

（2）按照标签是否连续，将监督学习分为：
- 分类问题
- 回归问题

## P3 机器学习算法的过程

## P4 没有免费午餐定理

没有免费午餐定理：任何一个预测函数，如果在一些训练样本上表现好，那么必然在另一些训练样本上表现不好，如果不对数据在特征空间的先验分布有一定的假设，那么表现好与表现不好的情况一样多。

没有免费午餐定理说明，如果不对特征空间的先验分布有假设，那么所有的算法表现都一样；机器学习的本质是通过有限的已知数据，在复杂的高维特征空间中预测未知样本的属性和类别。然而，我们并不知道未知样本到底在哪里，它的性质到底如何，因此再好的算法也存在犯错误的风险。没有免费午餐定理说明，没有放之四海而皆准的最好算法。因为评价算法的好坏涉及到对特征空间先验分布的假设，然而没有人知道特征空间先验分布的真实样子

## P5 总结
