title: Tensorflow的迭代方法
date: 2019-1-8
tags: [tensorflow]
categories: Machine Learning
description: 训练模型的迭代过程
---
[官方学习链接](https://developers.google.com/machine-learning/crash-course/reducing-loss/an-iterative-approach)

## 迭代过程
![Tensorflow迭代](/images/Tensorflow迭代过程.png)

　　将一群有标签样本传入模型训练，执行预测得到预测标签。通过损失函数比较实际标签与预测标签差异。再更新参数迭代直至发现损失可能最低的模型参数。
  
　　梯度下降法：梯度下降法首先选择一个任意起始值（起点）。然后计算损失曲线在起点处的梯度。（模长为最大方向导数）它可以让我们了解哪个方向距离目标“更近”或“更远”。值得注意的是，损失相对于单个权重的梯度（如下图）就等于导数。用梯度乘以一个称为学习速率（有时也称为步长）的标量，以确定下一个负梯度方向点的位置。来逼近最低点。当降到最低点时，模型即收敛。
  ![梯度下降法](/images/GradientDescentNegativeGradient.png)