---
tags: 视频编码 
mathjax: true 
title: In-Loop Filter / 融入QP信息的网络 
date: 2021-10-19
---

> 两篇paper
> A QP-ADAPTIVE MECHANISM FOR CNN-BASED FILTER IN VIDEO CODING
> One-for-all: An Efﬁcient Variable Convolution Neural Network for In-loop Filter of VVC

<!-- more -->

- 不同量化参数（QP）压缩的视频，其压缩噪声水平不同
- 不同的帧类型（FT），压缩噪声水平不同。
- 为单个QP/QP段或FT训练模型，消耗大量的资源训练大量模型，增加了视频编解码器的内存负担
- A practical convolutional neural network as loop filter for intra frame: QPmap

![](https://notes.sjtu.edu.cn/uploads/upload_23b54651e201bcbc4be324893286ff33.png)


- $\hat{\boldsymbol{y} }=\boldsymbol{w} *\{\boldsymbol{y} ; Q P\}+\boldsymbol{b}$


- $$
	\begin{aligned} \hat{\boldsymbol{y} } &=\boldsymbol{w}_{1} * \boldsymbol{y}+\boldsymbol{w}_{2} * Q P+\boldsymbol{b} \\ &=\boldsymbol{w}_{1} * \boldsymbol{y}+\left(\Sigma \boldsymbol{w}_{2} \cdot Q P+\boldsymbol{b}\right) \\ &=\boldsymbol{w}_{1} * \boldsymbol{y}+\boldsymbol{b}^{\prime}(Q P) \end{aligned}
	$$

- $\boldsymbol{b}^{\prime}(Q P)=\Sigma \boldsymbol{w}_{2} \cdot Q P+\boldsymbol{b}$
- QP被添加到线性函数的偏置上，没有反应出QP和滤波强度(权重)的关系，且只有第一层引入了QP信息
- QPmap不如QP-Separate/QP-Band models

## A QP-ADAPTIVE MECHANISM FOR CNN-BASED FILTER IN VIDEO CODING

### 问题建模

（忽略偏置bias）
时域卷积：$\boldsymbol{w} * \boldsymbol{y}=\hat{\boldsymbol{x } }$
频域乘积：$\mathcal{F}(\boldsymbol{w}) \mathcal{F}(\boldsymbol{y})=\mathcal{F}(\hat{\boldsymbol{x} })$
$\mathcal{F}(\hat{\boldsymbol{x} }) \approx \mathcal{F}(\boldsymbol{x})$

QP增加导致Qstep增加，引入噪声$\epsilon$

$\boldsymbol{w}^{\prime} *(\boldsymbol{y}+\boldsymbol{\epsilon})=\hat{\boldsymbol{x} }^{\prime}$

$\mathcal{F}\left(\boldsymbol{w}^{\prime}\right)(\mathcal{F}(\boldsymbol{y})+\mathcal{F}(\boldsymbol{\epsilon}))=\mathcal{F}\left(\hat{\boldsymbol{x} }^{\prime}\right)$

loss fuc：MSE

$\mathcal{L}=\mathbb{E}\left|\boldsymbol{x}-\hat{\boldsymbol{x} }^{\prime}\right|^{2}$

Parseval’s theorem

$\begin{aligned} \mathcal{L} &=\mathbb{E}\left|\mathcal{F}(\boldsymbol{x})-\mathcal{F}\left(\hat{\boldsymbol{x} }^{\prime}\right)\right|^{2} \\ &=\mathbb{E}\left|\mathcal{F}(\boldsymbol{x})-\mathcal{F}\left(\boldsymbol{w}^{\prime}\right)(\mathcal{F}(\boldsymbol{y})+\mathcal{F}(\boldsymbol{\epsilon}))\right|^{2} \\ & \approx \mathbb{E}\left|\mathcal{F}(\boldsymbol{x})-\mathcal{F}\left(\boldsymbol{w}^{\prime}\right)[\mathcal{F}(\boldsymbol{x}) / \mathcal{F}(\boldsymbol{w})+\mathcal{F}(\boldsymbol{\epsilon})]\right|^{2} \end{aligned}$
$$
\mathcal{F}\left(\boldsymbol{w}^{\prime}\right)=\underbrace{\mathcal{F}(\boldsymbol{w})}_{\text {org. filter } } \underbrace{\left[\frac{1}{1+|\mathcal{F}(\boldsymbol{w})|^{2} \mathcal{F}(\boldsymbol{n}) / \mathcal{F}(\boldsymbol{s})}\right]}_{\text {influence factor } }
$$

$$
\mathcal{F}(\boldsymbol{n})=\mathbb{E}|\mathcal{F}(\boldsymbol{\epsilon})|^{2} \text { and } \mathcal{F}(\boldsymbol{s})=\mathbb{E}|\mathcal{F}(\boldsymbol{x})|^{2}
$$
similar to Wiener deconvolution
- Wiener deconvolution：recover the original signal from the distorted signal by using the priors of the input signal, noise, and degradation function
- this solution：aims at making a speciﬁc ﬁlter adaptive to the changing quantization noise.

### 应用于CNN

从频域的角度来看，从CNN中提取的特征对应于输入图像在不同频率上的选择
作用在整个频带上的 $\boldsymbol{w}^{\prime}$ 可以被分解成作用在特定频带上的子滤波器 $w_{i}^{\prime}$ 

上式可写为

$\begin{aligned} \mathcal{F}\left(\boldsymbol{w}^{\prime}\right) &=\sum_{i} \mathcal{F}\left(w_{i}^{\prime}\right) \\ &=\sum_{i} \mathcal{F}\left(w_{i}\right)\left[\frac{1}{1+\left|\mathcal{F}\left(w_{i}\right)\right|^{2} \mathcal{F}\left(n_{i}\right) / \mathcal{F}\left(s_{i}\right)}\right] \end{aligned}$

第一项 $\mathcal{F}\left(w_{i}\right)$ 等价于CNN中的卷积核
第二项表示每个卷积核的 influence factor ，可以看作作用在特定子频带上

$\left|\mathcal{F}\left(w_{i}\right)\right|^{2} / \mathcal{F}\left(s_{i}\right)$ : 假设子带中$\left|\mathcal{F}\left(w_{i}\right)\right|^{2}$ 是常数，原始信号强度$\mathcal{F}\left(s_{i}\right)$ is also invariable in the task of adapting to different quantization noises，整体是常数 $k_i$

量化噪声强度 $\mathcal{F}(\boldsymbol{n})$ ：默认编码配置下，在所有频率上都与量化步长Qstep的平方成正比
分解子噪声 $\mathcal{F}\left(n_{i}\right)$载特定频率上与量化步长Qstep的平方成正比
$$
\frac{\left|\mathcal{F}\left(w_{i}\right)\right|^{2} }{\mathcal{F}\left(s_{i}\right)} \mathcal{F}\left(n_{i}\right) \approx k_{i} \mathcal{F}\left(n_{i}\right) \propto Q_{s t e p}^{2}
$$
trainable parameters θ  to represent the proportional relation- ship，then
$$
\mathcal{F}\left(\boldsymbol{w}^{\prime}\right) \approx \sum_{i} \mathcal{F}\left(w_{i}\right)\left[\frac{1}{1+\theta_{i} Q_{\text {step } }^{2} }\right]
$$

N ： the number of feature maps of CNN
number of parameters introduced by this method: $O\left(N^{2}\right)$, the same order as the number of the kernels

Inspired by DSC (depthwise convolution instead of standard con- volution), we apply the inﬂuence factor to the feature maps instead of the convolution kernels.

![](https://notes.sjtu.edu.cn/uploads/upload_325bde70ed8996168a743bc8d95c7486.png)

parameter quantity becomes $O\left(N\right)$, same order as the number of the biases (the QP map method).

- Qstep 的调整

VVC/HEVC: 

$Q_{s t e p}=2^{(Q P-4) / 6}$，$Q_{s t e p}^{2}=2^{(Q P-4) / 3}$

![](https://notes.sjtu.edu.cn/uploads/upload_18abe3cdcfd7d251d18cd079de1c589f.png)

- $\theta_{i}$ 不能小于等于 0

![](https://notes.sjtu.edu.cn/uploads/upload_e196c8ad1d5122ceeaac174effc64d46.png)

选择第二种直接截断的方法

### 实验结果

- 把这种方法应用到已经提出的这4个CNN上
- 网络加在DB和SAO之间，因为经过CNN增强后SAO ALF可能能节省一些比特
- CNN-被集成到 VTM-6.3
- DIV2K数据集，四个QPs包括22、27、32和37，测试用HEVC测试序列的第1帧

![只用luma训练模型，同时也能提升色度分量](https://notes.sjtu.edu.cn/uploads/upload_ed4a0a5646ca0cd4756c91f010e60835.png)

![解码复杂度](https://notes.sjtu.edu.cn/uploads/upload_305e44ab52fc39a81220ffd583e71115.png)

![与QPmap方法的对比](https://notes.sjtu.edu.cn/uploads/upload_15537338f437af0ddd716b8eda252373.png)



## One-for-all:  An  Efﬁcient  Variable  Convolution Neural  Network  for  In-loop  Filter  of  VVC

- 注意力模块 

给定离散值$x$，$x∈ Ω = [a, b]$, one-hot encoding 得向量

$\mathbf{v}_{\Omega}(x) \in \mathbb{R}^{m \times 1}$

$U \in \mathbb{R}^{C \times m}$ is a weight matrix

$\mathbf{M}=\sigma\left(U \mathbf{v}_{\Omega}(x)\right)$

$\sigma(x)=\log \left(1+e^{x}\right)$

![](https://notes.sjtu.edu.cn/uploads/upload_19cb97506db4a38e75b34b4ead57da6a.png)

![](https://notes.sjtu.edu.cn/uploads/upload_1ba48a0e32c2c84df6564fba3d7bae0b.png)

$I(x)=\left\{\begin{array}{l}0, \quad x \in[0, L] \\ 1+\left\lfloor\frac{x-L}{r}\right\rfloor, \quad x \in[L+1, R-1] \\ 1+\left\lfloor\frac{R-L}{r}\right\rfloor, \quad x \in[R, 63]\end{array}\right.$


x is QP value
I (x) is the index of the single high 1 bit
L = 20, R = 45 and r = 3. 

the shape of U is changed from (64, C ) to (10, C )

![](https://notes.sjtu.edu.cn/uploads/upload_9ef4aa964eb4b2d8e738dd4efb6acfce.png)

### 训练

![](https://notes.sjtu.edu.cn/uploads/upload_ed6f1b8343f0262e86f409df04379a20.png)

- CTU and frame-level RDO
- N = 1, D = 6,  3 × 3 convolutions C = 64
- DIV2K, All-intra conﬁguration with QP randomly selected from [21, 41]
- 40 videos of various resolutions (3840 × 2160, 1920 × 1080, 1280 × 720, 640 × 360), encoded under Low-delay P and Random-Access conﬁg- urations at QP=22, 27, 32, and 37. select one frame per ﬁve 
- 64 × 64 non-overlapping, remove the sub-images whose PSNR are more than 50.0 or less than 20.0
- Only use the Y-channel for training




- 训练 patch 分布不均匀，修改 loss func 为 focal MSE

	> 疑问：为何不直接构造分布均匀的数据集？

	![](https://notes.sjtu.edu.cn/uploads/upload_143d74670a6ed6c8f5e87a0abcd55450.png)
    - 首先用MSE训练一个VCNN，验证集计算增益率R
        - $R=1-\frac{l_{\text {rec } } }{l_{\text {init } } }$
        - $l_{\text {init } }=\|\mathbf{Y}-\mathbf{X}\|^{2}$ and $l_{r e c}=\|\hat{\mathbf{Y} }-\mathbf{Y}\|^{2}$

![](https://notes.sjtu.edu.cn/uploads/upload_e6838d51215601199e4c4b24c1221660.png)

有以下结论：

- 该网络在相对小QP上表现较好，而在较大QP上，如QP∈[42,44]，表现并不理想。
- 该网络在I帧上取得了最佳性能，在P和B帧上表现类似。
- 大多数有效数据的增益率低于10%，也就是说，增益率低的数据占了很大比例。所以期望网络能更多地关注具有低增益率的数据。

#### focal MSE

$\begin{aligned} L &=\frac{1}{N} \sum_{i=1}^{N}\left(\alpha_{q}^{(i)}+\alpha_{f}^{(i)}\right)\left(1-R^{(i)}\right)^{\gamma} l_{r e c}^{(i)} \\ &=\frac{1}{N} \sum_{i=1}^{N}\left(\alpha_{q}^{(i)}+\alpha_{f}^{(i)}\right) \frac{l_{\text {rec } }^{(i)^{1+\gamma} } }{l_{\text {init } }^{(i) } } \end{aligned}$

$\alpha_q$ : weighting factor over QP band
$\alpha_f$ : weighting factor over FT
$\gamma$ : focusing parameter

- ﬁrst utilize the following formula to adjust $\alpha_q$ $\alpha_f$ for each QP band and FT: $\alpha(t)=\frac{n \times(1-p(t))}{n-1}$
    - $n$ : the number of total types, i.e. n = 10 for QP and n = 3 for FT, 
    - $p(t)$ : the proportion of the data with type t. 
- then increase α(t) of type t where the network performs poorly.

![](https://notes.sjtu.edu.cn/uploads/upload_9b8235398f6d52404211def555c08153.png)

![](https://notes.sjtu.edu.cn/uploads/upload_57666646d6769dbb18f9715cd57159a1.png)







### 结果

- BD-rate

![](https://notes.sjtu.edu.cn/uploads/upload_2c88d6044b40b0ec1dddcbde15b5539e.png)

VCNN-S: separate qp model, without attention
VCNN-M: QPmap method

- Quality ﬂuctuation

![](https://notes.sjtu.edu.cn/uploads/upload_6462317026173b9ea3376d51739f28ad.png)

- 消融实验
    - 网络结构：复杂度和性能可以进一步权衡
    - 1）VCN-C32：C=32，D=6，N=1。
    - 2）VCN-D3：C=32，D=3，N=1。
    - 3）VCN-N3：C=64，D=6，N=3。
    - 4）VCNN：C=64，D=6，N=1

![](https://notes.sjtu.edu.cn/uploads/upload_c21fba20db9589b896b735b543afb452.png)

![](https://notes.sjtu.edu.cn/uploads/upload_29ca327de24ba74d082f4c45e5780c40.png)

- 注意力模块

![](https://notes.sjtu.edu.cn/uploads/upload_494e75bd6f31305c6cd665d6873d3150.png)

1) separate：网络结构与VCN-S相同，为每个QP段和FT分别训练，共4×3=12个
2) FT：为每个QP段单独训练模型，共4个
3) QP：为每个FT单独训练，共3
4) FT + QP：VCNN，1个模型

从数据集中随机选择1/5的数据来训练

> 疑问：为何 separate 表现最差？理论上separate应该表现最好

- focal MSE

![](https://notes.sjtu.edu.cn/uploads/upload_b7a6d06a0bcf37593311c8d8b71c8616.png)

- RFA

![](https://notes.sjtu.edu.cn/uploads/upload_38c0eb28bff9b9b66dffa31886851389.png)


- 以下实验都用来说明模型泛化性强

    - chroma

    ![](https://notes.sjtu.edu.cn/uploads/upload_b3f11be27e97b704f3cfa80121b40cbc.png)

    - Performance on other QPs
    
    ![](https://notes.sjtu.edu.cn/uploads/upload_de6a347d017bbd7ea3efd10ddef4cf1d.png)

    - HEVC

    apply our model trained on the VTM- 6.0 dataset directly to HM-16.9 without any ﬁne-tuning.

    ![](https://notes.sjtu.edu.cn/uploads/upload_7b42f1242eda5a0b29e9df5cd228a589.png)

    - 补充测试序列

    ![](https://notes.sjtu.edu.cn/uploads/upload_f925d0880e543aad122b64d6d0f15522.png)

- 计算复杂度

每种编码算法的计算复杂度是在RA配置下评估的。编码复杂度∆T的计算方法是

![](https://notes.sjtu.edu.cn/uploads/upload_bbb38e8af88f9c91b4c6b4b1303eb197.png)

![](https://notes.sjtu.edu.cn/uploads/upload_f84d929bc41d46cc80cde6c74f4cfc0e.png)


### 后续方向

- 解码复杂，无法在实时应用中实现。
    - 利用整数量化优化的网络。可以加快滤波的速度，避免在不同的硬件或软件平台上由于浮点运算而产生的不同的舍入误差
    - 训练一组简单和浅层的网络，编码器根据情况选择最优的，并将索引传送到解码端
    - 模型的推理依赖于第三方的深度学习框架，减少VVC编解码器和深度学习框架之间的互动造成的开销。

