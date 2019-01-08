# Batch-Normalization
## Batch Normalization说明
Batch Normalization作为最近一年来DL的重要成果，已经广泛被证明其有效性和重要性。
  《Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift》介绍了Batch Normalization。
   BatchNorm的功能是实现在深度神经网络训练过程中使得每一层神经网络的输入保持相同分布的。主要用于满足IID独立同分布假设：假设训练数据和测试数据是满足相同分布的，这是通过训练数据获得的模型能够在测试集获得好的效果的一个基本保障。
   随着深度学习中神经网络的深度加深，训练起来越来愈困难。收敛也越来越慢。这是深度学习十分本质的问题。很多论文都是解决这个问题的，比如ReLU激活函数，再比如Residual Network，BN本质上也是解释并从某个不同的角度来解决这个问题的。
## Internal Covariate Shift问题
  在论文中说明了Mini-Batch SGD相对于One Example SGD的两个优势：梯度更新方向更准确；并行计算速度快；（为什么要说这些？因为BatchNorm是基于Mini-Batch SGD的，所以先夸下Mini-Batch SGD，当然也是大实话）；然后吐槽下SGD训练的缺点：超参数调起来很麻烦。（作者隐含意思是用BN就能解决很多SGD的缺点）。
  接着引入`covariate shift`的概念：如果ML系统实例集合<X,Y>中的输入值X的分布老是变，这不符合IID假设，网络模型很难稳定的学规律，这不得引入迁移学习才能搞定吗，我们的ML系统还得去学习怎么迎合这种分布变化。对于深度学习这种包含很多隐层的网络结构，在训练过程中，因为各层参数不停在变化，所以每个隐层都会面临covariate shift的问题，也就是在训练过程中，隐层的输入分布老是变来变去，这就是所谓的`Internal Covariate Shift`，`Internal`指的是深层网络的隐层，是发生在网络内部的事情，而不是`covariate shift`问题只发生在输入层。
  BatchNorm的基本思想：能不能让每个隐层节点的激活输入分布固定下来呢？避免`Internal Covariate Shift`。
  BN思想源于图像处理中对输入图像进行白化(Whiten)处理：对输入数据分布变换到0均值，单位方差的正态分布——那么神经网络会较快收敛。
  推论：图像是深度神经网络的输入层，做白化能加快收敛，那么其实对于深度网络来说，其中某个隐层的神经元是下一层的输入，意思是其实深度神经网络的每一个隐层都是输入层，不过是相对下一层来说而已，那么能不能对每个隐层都做白化呢？这就是启发BN产生的原初想法，而BN也确实就是这么做的，可以理解为对深层神经网络每个隐层神经元的激活值做简化版本的白化操作。<br>
## BatchNorm的本质思想
   针对于传统神经元只对输入x进行标准化处理，例如减去均值，除标准差。以降低样本间的差异性。BN不仅仅只对输入层的输入数据x进行标准化，还对每个隐藏层的输入进行标准化.如下图所示。 <br>
  ![BN](https://img-blog.csdn.net/20170721163449112?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hpdGVzaWxlbmNl/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
- BN的基本思想：
  * 因为深层神经网络在做非线性变换前的激活输入值（就是那个x=WU+B，U是输入）随着网络深度加深或者在训练过程中，其分布逐渐发生偏移或者变动，之所以训练收敛慢，一般是整体分布逐渐往非线性函数的取值区间的上下限两端靠近（对于Sigmoid函数来说，意味着激活输入值WU+B是大的负值或正值），所以这导致反向传播时低层神经网络的梯度消失，这是训练深层神经网络收敛越来越慢的本质原因。<br>
  * 而BN就是通过一定的规范化手段，把每层神经网络任意神经元这个输入值的分布强行拉回到均值为0方差为1的标准正态分布，其实就是把越来越偏的分布强制拉回比较标准的分布，这样使得激活输入值落在非线性函数对输入比较敏感的区域，这样输入的小变化就会导致损失函数较大的变化，意思是这样让梯度变大，避免梯度消失问题产生，而且梯度变大意味着学习收敛速度快，能大大加快训练速度。<br>
**对于每个隐层神经元，把逐渐向非线性函数映射后向取值区间极限饱和区靠拢的输入分布强制拉回到均值为0方差为1的比较标准的正态分布，使得非线性变换函数的输入值落入对输入比较敏感的区域，以此避免梯度消失问题。<br>**
  
 **例**  假设某个隐层神经元原先的激活输入x取值符合正态分布，正态分布均值是-2，方差是0.5，对应下图中最左端的浅蓝色曲线，通过BN后转换为均值为0，方差是1的正态分布（对应上图中的深蓝色图形），意味着输入x的取值正态分布整体右移2（均值的变化），图形曲线更平缓（方差增大的变化）。<br>
  BN把每个隐层神经元的激活输入分布从偏离均值为0方差为1的正态分布通过平移均值压缩或者扩大曲线尖锐程度，调整为均值为0方差为1的正态分布。<br>
![正态分布](https://images2018.cnblogs.com/blog/1192699/201804/1192699-20180405225246905-37854887.png)  
  **目的：**当输入满足均值为0，输出为1的正态分布如下图所示，则输入值有64%的概率x其值落在[-1,1]的范围内，在两个标准差范围内，也就是说95%的概率x其值落在了[-2,2]的范围内。这对于实现深度学习快速收敛有十分明显的作用。<br>
  ![标准正态分布](https://images2018.cnblogs.com/blog/1192699/201804/1192699-20180405225314624-527885612.png)
  **BN对网络实现快速收敛的影响：**
  对于一个全连接神经网络，激活值x=WU+B,U是真正的输入，x是某个神经元的激活值，我们假设非线性函数是sigmoid，那么看下sigmoid(x)其图形：<br>
![sigmoid(x)](https://images2018.cnblogs.com/blog/1192699/201804/1192699-20180407143109455-1460017374.png"sigmoid(x)")
  在看以下sigmoid(x)的导函数，`G’=f(x)*(1-f(x))`，因为`f(x)=sigmoid(x)`在0到1之间，所以G’在0到0.25之间，其对应的图如下：  
![ Sigmoid(x)导数图](https://images2018.cnblogs.com/blog/1192699/201804/1192699-20180407142351924-124461667.png" Sigmoid(x)导数图")
   假设没有经过BN调整前x的原先正态分布均值是-6，方差是1，那么意味着95%的值落在了[-8,-4]之间，那么对应的Sigmoid（x）函数的值明显接近于0，出现梯度饱和的现象，梯度变化很慢。<br>
   **出现梯度饱和的原因：**如果取值接近0或者接近于1的时候对应导数函数取值，接近于0，意味着梯度变化很小甚至消失。<br>
   假设经过BN后，均值是0，方差是1，那么意味着95%的x值落在了[-2,2]区间内，很明显这一段是sigmoid(x)函数接近于线性变换的区域，意味着x的小变化会导致非线性函数值较大的变化，也即是梯度变化较大，对应导数函数图中明显大于0的区域，就是梯度非饱和区。<br>
   **通过BN，目前大部分Activation的值落入非线性函数的线性区内，其对应的导数远离导数饱和区，这样来加速训练收敛过程<br>**
  **注意**：在引进BN后，会出现非线性函数替换成线性函数效果。而线性函数对于神经网络的表达能力是没有意义的。因此，BN为了保证神经网络的有效性，对变换后的满足均值为0方差为1的x又进行了scale加上shift操作:`y=scale*x+shift`。对每个神经元增加了两个参数scale和shift参数，这两个参数是通过训练学习到的。意思是通过scale和shift把这个值从标准正态分布左移或者右移一点并长胖一点或者变瘦一点，每个实例挪动的程度不一样，这样等价于非线性函数的值从正中心周围的线性区往非线性区动了动。<br>
  **核心思想:** 找到一个线性和非线性的较好平衡点，既能享受非线性的较强表达能力的好处，又避免太靠非线性区两头使得网络收敛速度太慢。<br>
## 训练阶段如何做BatchNorm
  
