---
publish: true
---

[(26 封私信 / 63 条消息) 一文入门元学习（Meta-Learning）（附代码） - 知乎](https://zhuanlan.zhihu.com/p/136975128)
## 1. Introduction

通常在机器学习里，我们会使用某个场景的大量数据来训练模型；然而当场景发生改变，模型就需要重新训练。但是对于人类而言，一个小朋友成长过程中会见过许多物体的照片，某一天，当Ta（第一次）仅仅看了几张狗的照片，就可以很好地对狗和其他物体进行区分。

元学习Meta Learning，含义为学会学习，即learn to learn，就是带着这种对人类这种“学习能力”的期望诞生的。Meta Learning希望使得模型获取一种“学会学习”的能力，使其可以在获取已有“知识”的基础上快速学习新的任务，如：

- 让Alphago迅速学会下象棋
- 让一个猫咪图片分类器，迅速具有分类其他物体的能力

**需要注意的是，虽然同样有“预训练”的意思在里面，但是元学习的内核区别于迁移学习（Transfer Learning）**，关于他们的区别，我会在下文进行阐述。

接下来，我们通过对比机器学习和元学习这两个概念的要素来加深对元学习这个概念的理解。

![[Pasted image 20251113210409.png]]
在机器学习中，**训练单位是一条数据**，通过数据来对模型进行优化；数据可以分为训练集、测试集和验证集。在元学习中，训练单位分层级了，**第一层训练单位是任务，也就是说，元学习中要准备许多任务来进行学习，第二层训练单位才是每个任务对应的数据**。

二者的目的都是找一个Function，只是两个Function的功能不同，要做的事情不一样。机器学习中的Function直接作用于特征和标签，去寻找特征与标签之间的关联；而元学习中的Function是用于寻找新的f，新的f才会应用于具体的任务。**有种不同阶导数的感觉**。又有种**老千层饼的感觉，**你看到我在第二层，你把我想象成第一层，而其实我在第五层。。。

## 2. Meta Learning实施——以MAML为例

我们先对比机器学习的过程来进一步理解元学习。如下图所示，机器学习的一般过程如下：

- 设计网络网络结构，如CNN、RNN等；
- 选定某个分布来初始化参数；（以上其实决定了初始的f的长相，选择不同的网络结构或参数相当于定义了不同的f）；
- 喂训练数据，根据选定的Loss Function计算Loss；
- 梯度下降，逐步更新 ；
- 得到最终的f

对于人为设置的参数，我们叫做“超参数“。Meta Learning中希望把这些配置，如网络结构，参数初始化，优化器等由机器自行设计（注：此处区别于AutoML，迁移学习（Transfer Learning）[[【3】迁移学习]]和终身学习（Life Long Learning）[[【4】终身学习]] ），使网络有更强的学习能力和表现。

上文已经提到，**【元学习中要准备许多任务来进行学习，而每个任务又有各自的训练集和测试集】**。我们结合一个具体的任务，来介绍元学习和MAML的实施过程。

有一个图像数据集叫Omniglot：[https://github.com/brendenlake/omniglot](https://link.zhihu.com/?target=https%3A//github.com/brendenlake/omniglot)。Omniglot包含1623个不同的火星文字符，每个字符包含20个手写的case。这个任务是判断每个手写的case属于哪一个火星文字符。

如果我们要进行[N-ways](https://zhida.zhihu.com/search?content_id=118570471&content_type=Article&match_order=1&q=N-ways&zhida_source=entity)，[K-shot](https://zhida.zhihu.com/search?content_id=118570471&content_type=Article&match_order=1&q=K-shot&zhida_source=entity)（数据中包含N个字符类别，每个字符有K张图像）的一个图像分类任务。比如20-ways，1-shot分类的意思是说，要做一个20分类，但是每个分类下只有1张图像的任务。我们可以依据Omniglot构建很多N-ways，K-shot任务，这些任务将作为元学习的任务来源。构建的任务分为训练任务（Train Task），测试任务（Test Task）。特别地，每个任务包含自己的**训练数据、测试数据**，在元学习里，分别称为**Support Set和Query Set**。

**MAML的目的是获取一组更好的模型初始化参数（即让模型自己学会初始化）**。我们通过（许多）N-ways，K-shot的任务（训练任务）进行元学习的训练，使得模型学习到“先验知识”（初始化的参数）。这个“先验知识”在新的N-ways，K-shot任务上可以表现的更好。

![[Pasted image 20251113211341.png]]
