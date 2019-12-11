# Using-Binarized-Neural-Network-to-Slim-Models
This project will explore how the binary neural network can reduce the computation and the size of the model. Take MNIST and traffic signs recognition for example. The code is based on keras and runs on GPU.

本项目将探讨如何使用二值化神经网络优化模型，减少计算量并减少模型存储空间。本项目以mnist数据集和GTSRB（德国交通指示牌）为例。代码基于keras编写，支持GPU加速。

本项目主要包含四个部分：<br>
* 0.二值化神经网络简介；<br>
* 1.二值化神经网络计算原理；<br>
* 2.二值化神经网络训练算法；<br>
* 3.二值化神经网络识别手写数字；<br>
* 4.二值化神经网络识别交通指示牌。<br>


## 0.二值化神经网络简介<br>
为了将神经网络部署到诸如单片机这种算力有限的设备上，[二值化神经网络](https://arxiv.org/abs/1602.02830)被提出。二值网络是将权值W和隐藏层激活值二值化为1或者-1。通过二值化操作，模型的参数占用更小的存储空间（内存消耗理论上减少为原来的1/32倍，从float32到1bit）；同时利用位操作来代替网络中的乘加运算，大大降低了运算时间。由于二值网络只是将网络的参数和激活值二值化，并没有改变网络的结构。因此关注重点是如何二值化，以及二值化后参数如何更新。同时关注一下如何利用二进制位操作实现GPU加速计算的。

需要注意的是，二值化网络得到的权重值为二值（这样模型使用时的计算量将很小），但是训练过程中参与误差计算的梯度值是非二值化的。


## 1. 二值化神经网络计算原理<br>
二值化网络的计算重点在于梯度计算及梯度传递。

### 浮点数的二值化方法<br>
对任意一个 32 位浮点数x，其二值化方法为取其符号：x 不小于 0 时取 1，小于 0 时取 -1。
<p align="center">
	<img src="https://github.com/LeeWise9/Img_repositories/blob/master/%E4%BA%8C%E5%80%BC%E5%8C%96%E6%96%B9%E6%B3%951.png" alt="Sample"  width="300">
</p>

二值化操作如图所示：<br>
<p align="center">
	<img src="https://github.com/LeeWise9/Img_repositories/blob/master/%E4%BA%8C%E5%80%BC%E5%8C%96%E6%96%B9%E6%B3%952.png" alt="Sample"  width="500">
</p>

### 梯度传递计算方法<br>
虽然BNN 训练方法使用二值化的权值和激活值来计算参数梯度。但梯度不得不用其高精度的实际值，因为随机梯度下降（SGD）计算的梯度值量级很小，而且在累加过程中具有噪声，这种噪声是服从正态分布的，因此这种算子需要保持足够高的精度。此外，在计算梯度的时候给权值和激活值添加噪声具有正则化作用，可以防止过拟合。

符号函数sign 的导数为零，显然进无法行反向传播运算。因此，在反传过程中需要对符号函数进行松弛求解。

假设q 的梯度为：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/20180427221046833" alt="Sample"  width="40">
</p>

其中，C 为损失函数，已知 q 的梯度，那么 r 的梯度，即 C 对 r 的求导公式如下：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/20180427221418986" alt="Sample"  width="150">
</p>

其中 ，1|r|<=1  的计算公式为 Htanh，这也是函数变得可求导的原因，具体如下：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/2018042722191180" alt="Sample"  width="400">
</p>

即当r 的绝对值小于1时，r 的梯度等于 q 的梯度，否则 r 的梯度为 0 。可以用下图表示：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/20180427222533395" alt="Sample"  width="500">
</p>

梯度传递操作如图所示：<br>
<p align="center">
	<img src="https://github.com/LeeWise9/Img_repositories/blob/master/%E4%BA%8C%E5%80%BC%E5%8C%96%E6%96%B9%E6%B3%953.png" alt="Sample"  width="500">
</p>


## 2. 二值化神经网络训练方法<br>
二值化神经网络的训练和普通网络整体步骤相同，包括权值前传和误差后传。

### 前传<br>
前传的计算步骤如下图所示：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/20180428143438609" alt="Sample"  width="500">
</p>
（注意：二值化的变量均包含上角标b）

当计算层 k 不为最后一层时：<br>
	1. 对第 k 层的权值 Wk 做二值化操作；<br>
	2. 将二值化后的 Wk 与 第 k-1 层的二值化的激活值 a(k-1) 相乘得 sk；<br>
	3. 将 sk 做BatchNormalization 得 ak，注意 ak、sk 和 θk 都不是二值的；<br>
	4. 对 ak 做二值化。<br>
	k += 1<br>

### 后传<br>
后传的计算步骤如下图所示：<br>
<p align="center">
	<img src="https://img-blog.csdn.net/20180428144519222" alt="Sample"  width="500">
</p>
（注意：梯度值均非二值化）



## 3. 二值化神经网络识别手写数字<br>



## 4. 二值化神经网络识别交通指示牌<br>



