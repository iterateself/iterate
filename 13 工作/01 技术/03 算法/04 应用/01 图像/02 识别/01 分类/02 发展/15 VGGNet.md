# VGGNet

该网络所采用的 $3\times3$ 卷积核的思想是后来许多模型的基础


## 结构

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20190722/sslRwmdcvStR.png?imageslim">
</p>

> 图 4.7 VGG16 网络结构图

在原论文中的 VGGNet 包含了 6 个版本的演进，分别对应 VGG11、VGG11-LRN、VGG13、VGG16-1、VGG16-3 和 VGG19，不同的后缀数值表示不同的网络层数（VGG11-LRN表示在第一层中采用了 LRN 的 VGG11，VGG16-1表示后三组卷积块中最后一层卷积采用卷积核尺寸为 $1\times1$，相应的 VGG16-3表示卷积核尺寸为 $3\times3$）。

本节介绍的 VGG16 为 VGG16-3。图 4.7 中的 VGG16 体现了 VGGNet 的核心思路，使用 $3\times3$ 的卷积组合代替大尺寸的卷积（2个 $3\times3卷积即可与 $5\times5$ 卷积拥有相同的感受视野<span style="color:red;">为什么这么说？</span>），网络参数设置如表 4.5所示。

​表 4.5 VGG16网络参数配置：

|       网络层        |        输入尺寸         |             核尺寸              |        输出尺寸         |             参数个数              |
|:-------------------:|:-----------------------:|:-------------------------------:|:-----------------------:|:---------------------------------:|
|   卷积层 $C_{11}$   |  $224\times224\times3$  |      $3\times3\times64/1$       | $224\times224\times64$  |   $(3\times3\times3+1)\times64$   |
|   卷积层 $C_{12}$   | $224\times224\times64$  |      $3\times3\times64/1$       | $224\times224\times64$  |  $(3\times3\times64+1)\times64$   |
| 下采样层 $S_{max1}$ | $224\times224\times64$  |          $2\times2/2$           | $112\times112\times64$  |                $0$                |
|   卷积层 $C_{21}$   | $112\times112\times64$  |      $3\times3\times128/1$      | $112\times112\times128$ |  $(3\times3\times64+1)\times128$  |
|   卷积层 $C_{22}$   | $112\times112\times128$ |      $3\times3\times128/1$      | $112\times112\times128$ | $(3\times3\times128+1)\times128$  |
| 下采样层 $S_{max2}$ | $112\times112\times128$ |          $2\times2/2$           |  $56\times56\times128$  |                $0$                |
|   卷积层 $C_{31}$   |  $56\times56\times128$  |      $3\times3\times256/1$      |  $56\times56\times256$  | $(3\times3\times128+1)\times256$  |
|   卷积层 $C_{32}$   |  $56\times56\times256$  |      $3\times3\times256/1$      |  $56\times56\times256$  | $(3\times3\times256+1)\times256$  |
|   卷积层 $C_{33}$   |  $56\times56\times256$  |      $3\times3\times256/1$      |  $56\times56\times256$  | $(3\times3\times256+1)\times256$  |
| 下采样层 $S_{max3}$ |  $56\times56\times256$  |          $2\times2/2$           |  $28\times28\times256$  |                $0$                |
|   卷积层 $C_{41}$   |  $28\times28\times256$  |      $3\times3\times512/1$      |  $28\times28\times512$  | $(3\times3\times256+1)\times512$  |
|   卷积层 $C_{42}$   |  $28\times28\times512$  |      $3\times3\times512/1$      |  $28\times28\times512$  | $(3\times3\times512+1)\times512$  |
|   卷积层 $C_{43}$   |  $28\times28\times512$  |      $3\times3\times512/1$      |  $28\times28\times512$  | $(3\times3\times512+1)\times512$  |
| 下采样层 $S_{max4}$ |  $28\times28\times512$  |          $2\times2/2$           |  $14\times14\times512$  |                $0$                |
|   卷积层 $C_{51}$   |  $14\times14\times512$  |      $3\times3\times512/1$      |  $14\times14\times512$  | $(3\times3\times512+1)\times512$  |
|   卷积层 $C_{52}$   |  $14\times14\times512$  |      $3\times3\times512/1$      |  $14\times14\times512$  | $(3\times3\times512+1)\times512$  |
|   卷积层 $C_{53}$   |  $14\times14\times512$  |      $3\times3\times512/1$      |  $14\times14\times512$  | $(3\times3\times512+1)\times512$  |
| 下采样层 $S_{max5}$ |  $14\times14\times512$  |          $2\times2/2$           |   $7\times7\times512$   |                $0$                |
|  全连接层 $FC_{1}$  |   $7\times7\times512$   | $(7\times7\times512)\times4096$ |      $1\times4096$      | $(7\times7\times512+1)\times4096$ |
|  全连接层 $FC_{2}$  |      $1\times4096$      |        $4096\times4096$         |      $1\times4096$      |       $(4096+1)\times4096$        |
|  全连接层 $FC_{3}$  |      $1\times4096$      |        $4096\times1000$         |      $1\times1000$      |       $(4096+1)\times1000$        |

## 特性

- 整个网络都使用了同样大小的卷积核尺寸 $3\times3$ 和最大池化尺寸 $2\times2$。
- $1\times1$ 卷积的意义主要在于线性变换，而输入通道数和输出通道数不变，没有发生降维。<span style="color:red;">嗯，只是线性变换。</span>
- 两个 $3\times3$ 的卷积层串联相当于 1 个 $5\times5$ 的卷积层，感受野大小为 $5\times5$。同样地，3 个 $3\times3$ 的卷积层串联的效果则相当于 1 个 $7\times7$ 的卷积层。<span style="color:red;">这种等价关系是怎么得到的？</span>这样的连接方式使得网络参数量更小，而且多层的激活函数令网络对特征的学习能力更强。
- VGGNet 在训练时有一个小技巧，先训练浅层的的简单网络 VGG11，再复用 VGG11 的权重来初始化 VGG13，如此反复训练并初始化 VGG19，能够使训练时收敛的速度更快。<span style="color:red;">为什么可以这样训练？有什么理论依据吗？</span>
- 在训练过程中使用多尺度的变换对原始数据做数据增强，使得模型不易过拟合。<span style="color:red;">什么多尺度的变换。</span>



