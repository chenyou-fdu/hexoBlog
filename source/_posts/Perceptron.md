title: "感知机（Perceptron）"
date: 2015-03-07 20:46:26
tags: [机器学习, 感知机]
category: 机器学习
mathjax: true
---

## 损失函数
设为误分类点到超平面的总距离：$$\frac{1}{||\mathbf{w}||}\sum_{x_i\in{M}}y_i(w\cdot{x}+b)$$这里$||w||$是$\mathbf{w}$的$L_2$范数，$M$为误分类点集合。
这个总距离的求法：

设点$x_0$到超平面$S:\mathbf{w}\cdot\mathbf{x}+b=0$的距离为$d$。
设点$x_0$在平面$S$上的投影为$x_1$，则$\mathbf{w}\cdot\mathbf{x_1}+b=0$。
由于向量$\vec{x_0x_1}$与$S$平面的法向量$\mathbf{w}$平行，所以：
$$\|\mathbf{w}\cdot{\vec{x_0x_1}}\|=\|\mathbf{w}\|\|\vec{x_0x_1}\|=\sqrt{(w^1)^2+\cdots+(w^N)^2}d=\|\|\mathbf{w}\|\|d$$
又因为：
$$\mathbf{w}\cdot{\vec{x_0x_1}}=w^1(x^1_0-x^1_1)+\cdots+w^N(x^N_0-x^N_1)=w^1x^1_0+w^2x^2_0+\cdots+w^Nx^N_0-(-b)$$
所以：
$$\|\|\mathbf{w}\|\|d=\|w^1x^1_0+w^2x^2_0+\cdots+w^Nx^N_0+b=\|\mathbf{w}\cdot\mathbf{x}+b\|$$
这样就可以推出$d=\frac{1}{\|\|\mathbf{w}\|\|}\|\mathbf{w}\cdot\mathbf{x}+b\|$了。

因为我们接下来要对$\mathbf{w}$和$b$求导来获得损失函数的梯度，而$\|\|\mathbf{w}\|\|$是一个常数，所以我们忽略掉。那么就可以得到最终的损失函数了：
$$L(\mathbf{w},b)=-\sum_{x_i\in{M}}y_i(w\cdot{x_i}+b)$$
<!-- more  -->

## 随机梯度下降

使用随机梯度下降法（Stochastic Gradient Descent），极小化过程中不是一次使$M$中所有误分类点的梯度下降，而是一次随机选取一个误分类点使其梯度下降。
可以求得损失函数$L(\mathbf{w},b)$的梯度如下所示：
$$\nabla\_\mathbf{w}L(\mathbf{w},b)=-\sum_{x_i\in{M}}y_ix_i$$

$$\nabla\_bL(\mathbf{w},b)=-\sum_{x_i\in{M}}{y_i}$$

随机选取一个误分类点$(x_i,y_i)$，对$\mathbf{w}$和$b$进行更新（当$y_i(w_i+b)\le0$时）。
$$\mathbf{w}\longleftarrow\mathbf{w}+\eta{y_ix_i}$$$$b\longleftarrow{b+\eta{y_i}}$$
其中$\eta(0\le\eta\leq1)$是步长。这样，通过迭代可以期待损失函数$L(\mathbf{w},b)$不断减小，直到为0，**这样就形成了感知机学习算法的原始形式**。
标准梯度下降和随机梯度下降之间的区别以及收敛性的证明可以参考这篇博文[随机梯度下降](http://murongxixi.is-programmer.com/posts/40163.html)。

## 感知机学习算法的对偶式
基本思想：将$\mathbf{w}$和$b$表示为实例$x\_i$和标记$y\_i$的线性组合的形式，通过求解其系数而求得$\mathbf{w}$和$b$。
如果这里设$\mathbf{w}$和$b$的初值为$0$，那么最后学习到的$\mathbf{w}$和$b$可以分别表示为：
$$\mathbf{w}=\sum^N\_{i=1}\alpha\_iy\_ix\_i$$

$$b=\sum^N\_{i=1}\alpha\_iy\_i$$
这里$\alpha\_i\geq0,i=1,2,\cdots,N$，当$\eta=1$时，表示第$i$个实例点由于误分类而进行更新的次数。实例点更新次数越多，意味着它距离分离超平面越近，也就越难正确分类。也就是说这样的实例对学习结果影响最大。
于是有感知机的对偶式$f(x)=sign(\sum^N_{j=1}\alpha_iy_ix_i\cdot{x_i}+b)$，其中$\mathbf{\alpha}=(\alpha_1,\alpha_2,\cdots,\alpha_N)^T$，$\alpha$和$b$的初值都是$0$。
当出现误分类点时，对$\alpha$和$b$的更新为：
$$\alpha_i\longleftarrow\alpha_i+\eta$$

$$b\longleftarrow{b}+\eta{y_i}$$

可以提前计算Gram矩阵$G=[x\_i\cdot{x\_j}]\_{N\times{N}}$，以减少计算量。
使用对偶式的一个重要意义在于减少了需要优化求解的参数量，原始式需要求解的量和$\mathbf{w}$的维数相关，而对偶式需要优化求解的量只与训练数据的多少有关，这就减少了计算的时间，这个在支持向量机（SVM）中也得到了应用。而对偶式的解是否就是原始式的解的证明需要一些条件，原始式为线性关系是其中的一个条件，具体的证明过程这里略。

## Python实现
使用了`numpy`库，实现了原始式和对偶式的训练过程，使用的数据和书中一致。

```Python
from numpy import *
class Perceptron:
    def __init__(self,wLen,lRate = 1):
        self.weight = zeros((1,wLen))
        self.bias = 0
        self.learningRate = lRate
    def trainingPrimal(self,trainingValue,trainingLabel):
        while True:
            cnt = 0
            for t in range(trainingValue.shape[0]):
                temp = (((self.weight * trainingValue[t].T).sum()) + self.bias)*trainingLabel[t]
                if temp > 0:
                    cnt += 1
                else:
                    self.weight += self.learningRate*trainingLabel[t]*trainingValue[t]
                    self.bias += self.learningRate*trainingLabel[t]
            if cnt == 3:
                break
    def trainingDualForm(self,trainingValue,trainingLabel):
        GramMatrix = trainingValue*trainingValue.T
        tLen = trainingValue.shape[0]
        alpha = zeros((1,tLen)).T
        bCnt = 0
        while True:
            cnt = 0
            for i in range(tLen):
                temp = 0
                for j in range(tLen):
                    temp += GramMatrix[i,j]*trainingLabel[j]*alpha[j]
                temp += self.bias
                if(temp*trainingLabel[i] > 0):
                    cnt += 1
                    print i,temp*trainingLabel[i],alpha.T,"Good", self.bias
                else:
                    alpha[i] += 1*self.learningRate
                    self.bias += trainingLabel[i]*self.learningRate
                    print i,temp*trainingLabel[i],alpha.T,"Bad", self.bias
            print "end ",bCnt," loop"
            if cnt == 3:
                break
        for k in range(tLen):
            self.weight += alpha[k]*trainingValue[k]*trainingLabel[k,0]

if __name__ == '__main__':
    #training data
    trainingValue = matrix([(3,3),(4,3),(1,1)])
    trainingLabel = (matrix([1,1,-1])).T
    #model
    p = Perceptron(trainingValue.shape[1])
    p.trainingDualForm(trainingValue,trainingLabel)
    print p.weight
    print p.bias
```
