---
publish: true
---

> 学习链接：[GREYONE: Data Flow Sensitive Fuzzing | USENIX](https://www.usenix.org/conference/usenixsecurity20/presentation/gan)
> [[论文阅读] 02.清华张超老师Fuzzing总结 - GreyOne: Discover Vulnerabilities with Data Flow Sensitive Fuzzing - 知乎](https://zhuanlan.zhihu.com/p/446308018)


第一块是初始种子，它对Fuzzing的效率还是有很大影响的。如果你不给初始种子，它也会去测试，但是其效率比较低，很多学者去研究如何给一个好的初始种子，让Fuzzing更快地进入状态，更好地找到漏洞。

## 第一种是借用AI的方法  

基本思路是从程序的合法输入，网上爬取样本中学出一个模型，再用这个模型生成新的测试例，这样构造的初始种子相对来说更好。典型论文方法包括Skyfire、Learn&Fuzz、GAN、Neuzz等。

## **第二种是通过符号执行（Symbolic Execution）来辅助**  

这种辅助手段一般称为混合Fuzzing，其基本思路的核心还是Fuzzing来做，但Fuzzing有些代码过不去，比如一个复杂的数组检查，Fuzzing很难通过。对于这些过不去的分支，Drillers就提出用符号执行来辅助，遇到分支过不去的情况用符号执行来求解，并生成新的种子再丢给Fuzzing去通过分支，这是当时他们做CGC比赛的方案。符号执行和Fuzzing混合确实能提升过不去的分支。最近几年有进一步改进符号执行和Fuzzing的经典方法，比如QSYM、DigFuzz、HFL等。

## **第三种是基于静态分析和动态分析的**  

还有一些是基于静态分析、动态分析，以及去学习输入的规范，通过程序分析的技术手段去分析程序接受什么样的输入，再去指导测试例的生成。今年张老师他们有一篇针对Android服务的工作，也是这个思路，即FANS（USENIX Sec20）。

