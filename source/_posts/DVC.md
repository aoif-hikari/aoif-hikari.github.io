---
tags: end-to-end
mathjax: true 
title: End-to-end 视频编码框架： DVC 与 M-LVC
---

DVC: An End-to-end Deep Video Compression Framework
M-LVC: Multiple Frames Prediction for Learned VideoCompression

<!-- more -->

## DVC

- 单参考帧

![](https://notes.sjtu.edu.cn/uploads/upload_46ff30fd95be45de8b030b6e04d8a96f.png)


### 运动估计


- 使用CNN模型估计光流(运动信息)

spynet

![](https://notes.sjtu.edu.cn/uploads/upload_9c0468d02e7397d0e6f3dc09871ecd26.png)


- 运动编码网络，auto-encoder结构，压缩光流，量化

![](https://notes.sjtu.edu.cn/uploads/upload_2de6cc62a832d95b26953a9d25a9c1d1.png)

> End-to-end Optimized Image Compression (google iclr_17)
> ![](https://notes.sjtu.edu.cn/uploads/upload_9344269049815a0ddc301862bc90b6bc.png)
> ![](https://notes.sjtu.edu.cn/uploads/upload_9bfec99932d669a2230dedf4d4fbf20e.png)
> 使用了GDN(generalized divisive normalization)进行归一化处理。类似一般CNN网络中的Batch Normalization作用，可以捕捉图像的统计特性，并将其转换为高斯分布。
> 
> 量化：假设量化区间为1，即为将y量化为它最近的整数
> $$
> \hat{y}_{i}=q_{i}=\operatorname{round}\left(y_{i}\right)
> $$
> $\hat y$ 的边缘密度(离散概率分布)可以表示为
> $$
> P_{q_{i} }(n)=\int_{n-\frac{1}{2} }^{n+\frac{1}{2} } p_{y_{i} }(t) \mathrm{d} t, \quad \text { for all } n \in \mathbb{Z}
>$$
> 即 $p_{y_i}(t)$ 是 $y$ 的概率密度函数，由于四舍五入，在某一整数的（-0.5，0.5）区间内都会量化为该整数，则通过积分的形式计算这一区间内的数值的出现概率，得到量化后的整数的出现概率。

> 优化目标是网络的率失真性能 $R+\lambda D$，损失函数设计为
> $$
>L\left(g_{a}, g_{s}, P_{q}\right)=-E\left[\log _{2} P_{q}\right]+\lambda E[d(z, \tilde{z})]
>$$
> R, is lower-bounded by the entropy of the discrete probability distribution of the quantized vector, $H [P_q]$
> 
> 涉及到量化问题，但是量化会导致不可微分，阻断反向传播优化。在训练过程中添加（-0.5，0.5）范围的均匀噪声$\tilde{\boldsymbol{y} }=\boldsymbol{y}+\Delta \boldsymbol{y}$
> 即采用这种形式近似可微以用于反向传播的优化。在推理过程中，则依旧使用round 函数进行四舍五入（因为不用进行优化了）。
> 
> ![](https://notes.sjtu.edu.cn/uploads/upload_33abb0c4538dfa3a60b30c3ed4200fd9.png)


### 运动补偿

![](https://notes.sjtu.edu.cn/uploads/upload_086a9f9ef0f9f6e3cd6cecfa2d486ba1.png)

![](https://notes.sjtu.edu.cn/uploads/upload_63e81feb7ecf9972b1a543962cddbb62.png)


给定前一重建帧 $\hat x_{t-1}$ 和解码的运动矢量 $\hat v_t$，运动补偿网络将获得预测帧 $\bar x_t$。

首先基于运动信息 $\hat v_t$ 将 $\hat x_{t-1}$ warp到当前帧，warp的帧有伪影，所以将warped frame 和 $\hat v_t$，$\hat x_{t-1}$共同作为运动补偿网络输入。

该方法是像素级的运动补偿方法，避免了传统的基于块的运动补偿方法中的块效应，不需要环路滤波模块。

### 变换、量化

- 残差编码网络

原始帧 $x_t$ 和预测帧 $\bar x_t$ 之间的残差信息 $r_t$ 由残差网络编码

>Variational image compression with a scale hyperprior (google iclr_18)
![](https://notes.sjtu.edu.cn/uploads/upload_d50ec3ca86a40a4252def91730390001.png)


- 量化

> End-to-end Optimized Image Compression (google iclr_17)
>
> 训练阶段通过加入均匀噪声来代替量化运算，推理阶段运动信息和残差直接四舍五入取整量化

### 熵编码

- 量化运动信息和量化残差编码为比特
- 使用比特率估算网络获得中每个符号的概率分布，估算比特数成本

> Variational image compression with a scale hyperprior (google iclr_18)

## M-LVC 

- 多参考帧

![](https://notes.sjtu.edu.cn/uploads/upload_988f0dfe898c62afb33b944e3a430dd4.png)

### 运动估计与预测

- 运动估计网络（ME-Net）使用 FlowNet2.0 生成光流MV （$v_t$）

![](https://notes.sjtu.edu.cn/uploads/upload_2b161a3a6260e3c4c4db1eac99068a7c.png)

- 采用MV预测网络（MAMVP-Net）来预测当前的MV($\bar v_t$)

![](https://notes.sjtu.edu.cn/uploads/upload_e276dfa8ed49aa9ed1055da91e0f9bc7.png)

采用先前多个重构的MV对当前MV进行预测，上面网络图中采用先前三个重构MV进行预测。首先，对先前每个MV进行金字塔特征提取，见（a）
$$
\left\{f_{\hat{v}_{t-i} }^{l} \mid l=0,1,2,3\right\}=H_{m f}\left(\hat{v}_{t-i}\right), i=1,2,3
$$
其次，考虑到先前重构MV有错误，对抽取的金字塔特征进行warp
$$
\begin{aligned}
f_{\hat{v}_{t-3} }^{l, w} &=\operatorname{Warp}\left(f_{\hat{v}_{t-3} }^{l}, \hat{v}_{t-1}^{l}+\operatorname{Warp}\left(\hat{v}_{t-2}^{l}, \hat{v}_{t-1}^{l}\right)\right) \\
f_{\hat{v}_{t-2} }^{l, w} &=\operatorname{Warp}\left(f_{\hat{v}_{t-2} }^{l}, \hat{v}_{t-1}^{l}\right), l=0,1,2,3
\end{aligned}
$$
再次，利用金字塔网络从粗到细预测当前MV，见(b)。

- 使用MVD编码器/解码器网络（google iclr_17）对$v_t$和$\bar v_t$之差$d_t$进行编码
    - $d_t$ -> $m_t$ -> $\hat m_t$
    - 使用google iclr_17中的CNN估计$\hat m_t$的分布，熵编码为比特流
    
- 由于解码后的$\hat{d}_t$包含量化误差，使用MV优化网络（MV Refine-Net）来减少量化误差并提高质量。将优化后的MV存在解码的MV缓冲器中，用于下一帧编码。

![](https://notes.sjtu.edu.cn/uploads/upload_b1d75a450aed0ed8e9cc32f087df4f50.png)


### 运动补偿

- 重构MV后，使用运动补偿网络（MMC-Net）获得预测帧$\bar x_t$，MMC-Net可以通过使用多个参考帧来生成更准确的预测帧

![](https://notes.sjtu.edu.cn/uploads/upload_229eb93048d2fae3843cde7a28df12c1.png)


### 残差

- 运动补偿后，残差编码网络（google 18_iclr，同DVC）对原始帧$x_t$和预测帧$\bar{x}_t$之间的残差$r_t$进行编码。
    - $r_t$ -> $y_t$ -> $\hat y_t$
    - 使用google 18_iclr中的CNN估计$\hat y_t$的分布，熵编码为比特流
    
- 解码的$\hat{r}'_t$包含量化误差，因此使用残差优化网络（Residual Refine-Net）来减少量化误差并提高质量。

![](https://notes.sjtu.edu.cn/uploads/upload_b771cb5e9d315bc82af335dc4b7c0874.png)



### 重建帧

- 在优化残差后，将加到预测帧上来获得重构帧。然后将重构帧缓存在解码的帧缓冲器中以用于下一帧编码。

### Training Strategy

#### Loss Function.
$$
J=D+\lambda R=d\left(x_{t}, \hat{x}_{t}\right)+\lambda\left(R_{m v d}+R_{r e s}\right)
$$

- $d(x_{t}, \hat{x}_{t})$ 使用 MSE
- $R_{m v d}, R_{r e s}$ 别表示用于编码MVD和残差的比特率。在训练过程中，不进行实际编码，而是根据相应的$\hat{m}_t$和$\hat{y}_t$的熵来估计比特率。分别使用google 17_iclr和google 18_iclr中的CNNs来估计$\hat{m}_t$和$\hat{y}_t$的概率分布，然后得到相应的熵。
- 训练过程中通过添加均匀噪声来代替量化运算(google 17_iclr)。















