---
tags: VVC
mathjax: true 
title: Video Coding / ALF
date: 2021-10-20 
---

> ALF, VVC中增益最大的单项工具。

<!--more-->

ALF思想：维纳滤波。

维纳滤波是一种基于最小均方误差准则、对平稳过程的最优估计器，其输出与期望输出之间的均方误差为最小。

下图为ALF在VVC中位置。

<img src="/img/VVC/alf.png" style="zoom:67%;" />

ALF使用如下所示两种钻石形状的滤波器，5x5大小的适用于色度分量，7x7适用于亮度分量。其中滤波系数平均值为128。

<img src="/img/VVC/alfshape.png" style="zoom: 33%;" />

`sh_alf_enabled_flag`

`sh_alf_cb_enabled_flag`

`sh_alf_cr_enabled_flag`

## ALF解码端滤波过程

### 1. 块分类

> 对不同的小块使用不同的滤波器，因此需将这些小块进行分类。

对于**亮度**，每个4x4小块被分为25类，每类有相应的滤波器。类别$C$计算规则如下。

$C=5D+\hat A$

其中$D$和$A$分别表示当前块的$Direction$和$Activity$，计算之前需要当前**块**的水平、垂直和两个对角方向的gradient。

<img src="/img/VVC/block-gradient.png" style="zoom:67%;" />

$(i, j)$为当前4x4块的左上角坐标。4x4子块的梯度等于当前4x4块周围补两圈共64个像素的梯度之和。其中像素梯度用1-D拉普拉斯算子计算

<img src="/img/VVC/pixel.png" style="zoom:67%;" />

为了减少计算量，64个像素的梯度值不必都计算，而是通过下图方式下采样，只计算标注的像素的梯度。该过程是在一个10x10大小的窗口中进行。

<img src="/img/VVC/down.png" style="zoom:67%;" />

#### 计算Direction

- 水平垂直方向上梯度的最大最小值：


<img src="/img/VVC/h.png" style="zoom: 67%;" />

​		对角线方向上梯度的最大最小值：

<img src="/img/VVC/d.png" style="zoom:67%;" />

- 根据上述四个值以及阈值$t_1$和$t_2$设置D的值，规则如下
  <img src="/img/VVC/setd.png" style="zoom: 80%;" />

step1中子块被划分为纹理，step3中子块被划分为强/弱水平或垂直，step4中子块被划分为强/弱对角线方向。

#### 计算Activity

- 计算$A$：

<img src="/img/VVC/A.png" style="zoom: 80%;" />

- 将A映射为值在0—4之间的$\hat A$

<img src="/img/VVC/map.png"  />

<img src="/img/VVC/Q.png"  />

### 2. 几何变换（Geometric transformations）

> 其目的是使不同块的方向性一致，以减少ALF类的数量，反过来也减少了过滤系数。应用几何变换可以使具有水平边缘的4×4块和具有垂直边缘的4×4块都具有相同的方向性 ，具体进行哪一种翻转取决于第一步中计算得到的gradient。

几何变换包括三种，对角翻转Diagonal、垂直翻转Vertical flip、旋转Rotation，以7x7的块为例如下图：

<img src="/img/VVC/flip.png" style="zoom: 50%;" />



几何变换规则如下表：

<img src="/img/VVC/trans.png" style="zoom: 50%;" />

### 3. 滤波

#### 线性滤波

在$(x, y)$处像素值为$R(x,y)$，滤波后像素值$\tilde{R}(x,y)$
$$
\begin{aligned}
\tilde{R}(x, y)=&\left[\sum _ { i = 0 } ^ { N - 2 } c _ { i } \left(R\left(x+x_{i}, y+y_{i}\right)+R\left(x-x_{i}, y-\right.\right.\right.\left.\left.\left.y_{i}\right)\right)+c_{N-1} R(x, y)+64\right]>>7
\end{aligned}
$$
化简得
$$
\begin{aligned}
\tilde{R}(x, y)=& R(x, y)+\left\{\left[\sum _ { i = 0 } ^ { N - 2 } c _ { i } \left(R\left(x+x_{i}, y+y_{i}\right)-\right.\right.\right.R(x, y))+\sum_{i=0}^{N-2} c_{i}\left(R\left(x-x_{i}, y-y_{i}\right)-\right.R(x, y))+64]>>7\}
\end{aligned}
$$
对于7x7和5x5的块，N分别为13和7。

#### 非线性自适应滤波

> 引入clip function来减少临近样本和当前样本差异过大时的影响。

<img src="/img/VVC/nonliner.png"  />
$$
\begin{array}{c}
f_{i}=\min \left(b_{i}, \max \left(-b_{i}, R\left(x+x_{i}, y+y_{i}\right)-R(x, y)\right)\right)+\min \left(b_{i}, \max \left(-b_{i}, R\left(x-x_{i}, y-y_{i}\right)-R(x, y)\right)\right)
\end{array}
$$
$min(y, max(-y, x))$对应于$Clip3(-y, y, x)$

$b_i$： clipping parameter，取决于$BitDepth$和$clipidx$，如下表。

<img src="/img/VVC/bi.png"  />

相关flag：

`alf_luma_clip_flag`

`alf_luma_clip_idx\[ sfIdx ][ j ]：表示传输的第sfIdx个滤波器的第  j 个系数的clipidx。`

`alf_chroma_clip_flag`

`alf_chroma_clip_idx`

## 减少行缓冲区的边界过滤过程

<img src="/img/VVC/linebuffer.png"  />

如果没有行缓冲区边界处理，需要使用DBF和SAO过滤后的B行到K行的样本。
在下层CTU可用之前，DBF和SAO滤波器不能应用于A至D行。因此，如果没有行缓冲区边界处理，ALF就不能应用于E到H行的样本，直到下层CTU可用。
除了A到D行之外，E到K的7个luma行必须存储在行缓冲器中用于luma ALF。同样地，4个额外的色度行将存储在行缓冲器中用于色度ALF
 为了计算与行缓冲区边界相邻的样本(E行)梯度值，padding，即D行的样本被替换为E行的样本。
行缓冲区边界另一侧的所有样本梯度值都被设置为0，减少了样本梯度的总和，需要对活动性A进行缩放：

<img src="/img/VVC/scalea.png"  />

当要过滤的样本的滤波形状越过行缓冲区边界时，对称样本填充。白色为待滤波像素，灰色为对称padding后的像素

<img src="/img/VVC/sy.png" style="zoom:80%;" />

当要过滤的样本位于最接近行缓冲区边界的一行时，二维滤波器相当于一个水平滤波器，引入明显的视觉伪影，将滤波强度降低8倍可以将这些伪影降到最低。公式中右移7相应变为右移10。

<img src="/img/VVC/10.png" style="zoom: 67%;" />

## CCALF

> 用亮度样本修正色度分量，提供近1%的BD-Rate增益

<img src="/img/VVC/ccalf.png" style="zoom:67%;" />

CCALF滤波器系数值必须是2的次幂，{0, 1, 2, 4, 8,16, 32, 64}，方便移位运算。

相关flag：

`Slice级：`

`sh_alf_cc_cb_enabled_flag`

`sh_alf_cc_cr_enabled_flag`

`CTB级：`

`alf_cc_cb_filter_signal_flag`

`alf_cc_cb_filters_signalled_minus1`

`alf_cc_cb_mapped_coeff_abs`

`alf_cc_cb_coeff_sign`

## 编码端ALF

### 系数计算过程

> 维纳滤波器的基本原理是找到一个合适的单位冲激响应,使得加入了随机噪声的信号通过滤波器后,得到的信号与原始信号的均方误差MSE最小。

设$o[i]$为原始未压缩图像的像素值,其中$i∈I$为坐标矢量,$I$是所有被该滤波器进行滤波的像素点坐标的集合;

$r[i]$为经过压缩后未滤波的像素值,也就是经过DBK和SAO后的像素值,相比原始像素值引入了随机噪声;

$f[i]$为滤波后的像素值。

$c_n$为滤波系数，$p_n$是该滤波器系数相对滤波器中心位置的偏移值。

可以求得滤波后的像素值与原始像素值的均方误差

<img src="/img/VVC/alfc1.png" style="zoom:67%;" />

MSE对各系数求偏导

<img src="/img/VVC/alfc2.png" style="zoom:67%;" />

为使MSE最小，令其对各系数的偏导数为0，有

<img src="/img/VVC/alfc3.png" style="zoom:67%;" />

将上式写成矩阵形式，

<img src="/img/VVC/alfc4.png" style="zoom: 50%;" />

令

<img src="/img/VVC/alfc5.png" style="zoom: 67%;" />

<img src="/img/VVC/alfc6.png" style="zoom: 67%;" />



有

<img src="/img/VVC/alfc7.png" style="zoom:67%;" />

令

<img src="/img/VVC/alfc8.png" style="zoom:67%;" />

R矩阵由未滤波的像素值组成，得

<img src="/img/VVC/alfc9.png" style="zoom:67%;" />

$T$和未滤波像素值有关，$v$和未滤波像素值及原始像素值有关。求解该方程组就可以得到相应的滤波系数。

对于 ALF 来说，由于其卷积核是中心对称的，所以在求解过程中可以将相同系数的像素值先合并再计算，即

<img src="/img/VVC/alfc10.png" style="zoom: 67%;" />

### 快速算法求滤波后失真

编码端需要多次RDO，需要求得滤波后的误差，复杂度高。所以算滤波后MSE实际并没有进行滤波，而是采用快速算法。

根据 MSE 的定义有

<img src="/img/VVC/alfc11.png" style="zoom:67%;" />

或采用矩阵方式

<img src="/img/VVC/alfc12.png" style="zoom:67%;" />

由于相应像素集对应的 T 和 v 可以快速求出，所以新的 MSE 也可以快速求出，而不需要进行真正的滤波操作。

进一步简化可得

<img src="/img/VVC/alfc13.png" style="zoom:67%;" />

这里有一部分的计算和其他模块的计算重复了。即不使用 ALF，未滤波时的MSE。上述公式中已经包含了这一部分，推导见下：

由MSE的定义，

<img src="/img/VVC/alfc14.png" style="zoom:67%;" />

矩阵计算版：

<img src="/img/VVC/alfc15.png" style="zoom:67%;" />

这里实际上可提取出MSE non−filtered这一项，也就是不使用 ALF 时的 MSE，而这一项前面已经求得，因此只需要求前面那部分即可。

对于维纳滤波的系数来说，由于有 $T \hat c = \hat v$，所以也可简化为

<img src="/img/VVC/alfc16.png" style="zoom:67%;" />

### 产生新滤波器的过程

如何对每一帧产生new filter set：

- 对每一帧，通过merge ALF on的CTBs的数据产生一个亮度filter set。在第一轮迭代中，假设所有CTBs都ALF on。


- 对每个CTB用产生的filters和自身数据做RDO，决定是否使用ALF。


- 以上步骤迭代四次。


产生filter set具体步骤：

- 首先对25类分别算一个滤波器。


- 在每次迭代中，通过合并两个滤波器，将滤波器的数量减少1。为了确定哪两个滤波器应该被合并，对于剩下的每一对滤波器，编码器分别通过合并两个滤波器和相应的统计数据重新设计一个滤波器。使用重新设计的滤波器，然后估计失真度，编码器合并失真度最小的一对。


- 得到25个滤波器组，第一组有25个滤波器，第二组有24个滤波器，以此类推，直到第25组包含一个滤波器。


- 选择RDcost最小的一组


产生具体一个filter：

- 用CTB的数据解Wiener-Hopf方程，得coeff。clipidx和coeff迭代计算直到方差不再降低。


- 在每轮迭代中，clipidx一个一个更新，从$d_0$到$d_{N-2}$，每个clipidx有三种选择：保持不变，加1，减1，对每一种算coeff和distortion，选方差最小的。第一轮迭代clipidx初始化为2。合并两个滤波器时，clipidx取对应的均值。

## syntax

滤波相关参数放在APS中。

一个ALF APS最多包括1 luma filter set 和8 chroma filters，为每个色度分量最多包含四个CC-ALF滤波器，还有clipidx。1 filter set 最多包括25个滤波器（对应25种类），不同类别的滤波器参数可以merge，所以实际一般少于25个。

<img src="/img/VVC/alfsyntax1.png"  />

`alf_luma_num_filters_signalled_minus1：加1表示对亮度分量传输的滤波器数量，其值在[0,24]间。`

`alf_luma_coeff_delta_idx[ filtIdx ]：filtIdx 所指定类别对应的滤波器增量的索引。即表明了类别merge后每类所对应的滤波器。`

`alf_luma_coeff_abs\[ sfIdx ][ j ]：表示传输的第  sfIdx  个滤波器的第  j 个系数 的绝对值。其值在[0,128]间。`
`alf_luma_coeff_sign\[ sfIdx ][ j ]：表示传输的第  sfIdx  个滤波器的第  j 个系数 的符号。`

`alf_luma_clip_idx\[ sfIdx ][ j ]：表示传输的第  sfIdx  个滤波器的第  j 个系数的clipidx。`

<img src="/img/VVC/alfsyntax2.png"  />
`alf_chroma_num_alt_filters_minus1：加  1 表示色度可使用的滤波器数量， 其值在[0,7]间。`
`alf_chroma_coeff_abs\[ altIdx ][ j ]：表示的第  sfIdx  个色度滤波器的第  j 个系 数的绝对值。其值在[0,128]间。`
`alf_chroma_coeff_sign\[ altIdx ][ j ]：表示的第  sfIdx  个色度滤波器的第  j 个 系数的符号。`
`alf_chroma_clip_idx\[ altIdx ][ j ]：表示第  sfIdx  个色度滤波器的第  j 个系数的clipidx的索引。`



APS可用数量在slice级别控制（`sh_num_alf_aps_ids_luma`），最多可用7个。

除了7个APS，还提供了16个预训练好的固定滤波器，APS种类为索引为0~15。CTB级可以控制ALF开关，指定所使用的滤波器组。一个luma CTB可以使用为当前slice计算的aps或为已经编码的slice计算的aps之一

<img src="/img/VVC/alfeg.png" style="zoom:;" />



## 结果

anchor为指定技术off

<img src="/img/VVC/alfres.png" style="zoom: 67%;" />

尽管ALF编码器被设计为改善客观质量，但也可以观察到主观质量的改善。在图12的例子中，ALF同时减少了振铃伪影（用黑色圆圈标记）和阻塞伪影（用白色圆圈标记）。

<img src="/img/VVC/alfres2.png" style="zoom:;" />

CC-ALF的BD-rate显示出色度的增益和luma的损失，因为CC-ALF花费的额外比特被用来提高色度质量，而luma质量几乎保持不变。CC-ALF的测试结果显示在下表中，包含三个部分。第一部分显示了编码器以PSNR增益为目标，内容为YCbCr色彩空间，采用ITU-R BT.1886光电子传递函数（OETF）的情况下的结果。在第二部分中，编码器再次以PSNR增益为目标，而内容是在RGB色彩空间中的ITU-R BT.1886 OETF。第三部分与PQ 4:2:0内容有关，显示了编码器以加权PSNR指标为目标的结果。

<img src="/img/VVC/ccalfres.png" style="zoom:80%;" />

<img src="/img/VVC/ccalfres2.png" style="zoom:80%;" />

## 代码

<img src="/img/VVC/codealf.png"  />

