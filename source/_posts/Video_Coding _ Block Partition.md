---
tags: video coding
mathjax: true 
title: Video Coding / Block Partition
date: 2021-11-16
---

> Block Partition，Video Coding 的基础。

<!-- more -->

## H.264

- macroblock 16x16, 在一个MB内划分PU, TU
    - I macroblock(intra)
        - PB: luma 4x4, 16x16; chroma: 16x16
    - P/B macroblock(inter)
        - PB: 2Nx2N, Nx2N, 2NxN, NxN
            - ![macroblock 进一步 PU 划分](https://notes.sjtu.edu.cn/uploads/upload_35672750ff82e0032d687554f6eea5d0.png)
    - TB within an MB: 4x4 DCT2; 8x8 DCT2 in FRExt

![大分辨率序列上效率低](https://notes.sjtu.edu.cn/uploads/upload_8ef1bf563d8fcaa5ad45d30c504ababd.png)

## H.265

- CTU: 64X64 ---quad-tree---> CU: 8x8, 16x16, ... 64x64
    - ![](https://notes.sjtu.edu.cn/uploads/upload_8e63500a385f17b8b3c4874752b4b2ca.png)
    - CU --> PU
        - intra: 2Nx2N, NxN(N=4, 8, 16, 32)
        - inter: 2Nx2N, Nx2N, 2NxN, NxN, AMP
            - ![](https://notes.sjtu.edu.cn/uploads/upload_5ba2983cd9e4f1ed54ecda83c73c32af.png)

    - CU ---Residual quad tree---> TU
        - 4x4, 8x8, 16x16, 32x32 DCT-2
        - 4x4 DST-7 for intra 4x4 luma block

![](https://notes.sjtu.edu.cn/uploads/upload_b3d3c8a256459361a7095512c8c9faa4.png)


## H.266

- CTU 128x128 ----QT + MTT(multi-type-tree)---> CU: 8x8, 16x16, ... 128x128
    - ![MTT](https://notes.sjtu.edu.cn/uploads/upload_80c198b7a7bef214264f455a662378f1.png)
    - ![左下的CTU经历了三次划分，(编码端决策)复杂度已经很高](https://notes.sjtu.edu.cn/uploads/upload_9ff1804a48e3deec292ad5d2ef10c62b.png)

    - CU划分灵活，取消PU
    - CU --->TU(取消RQT划分)
        - ISP(intra subpartition)
        - SBT(sub-block transform)

![](https://notes.sjtu.edu.cn/uploads/upload_c0a509e00ae2037086d4bfe0a645fbae.png)

### CTU --> CU

#### QT-MTT
- ![](https://notes.sjtu.edu.cn/uploads/upload_964095e103d8c1e6d2cd3d4e5a93e21d.png)
- ![](https://notes.sjtu.edu.cn/uploads/upload_09d6c5324e183c5d382b71be2cfcde85.png)
- ![](https://notes.sjtu.edu.cn/uploads/upload_6397fadb947252dccda1e38fcfcfe2da.png)

> ![](https://notes.sjtu.edu.cn/uploads/upload_6c8e028c48d9fd597c43afc8d44b8b70.png)

- ![](https://notes.sjtu.edu.cn/uploads/upload_16e8b9e266046d5aeccea4c27d223c69.png)

![](https://notes.sjtu.edu.cn/uploads/upload_dbc58176795f865ac53bbe7d309e8db3.png)

> 划分中止条件(VVC)：划分到最小 CU 块；达到规定的最大二级树的划分深度
> ![](https://notes.sjtu.edu.cn/uploads/upload_c27c39236f4094d5ceb16ad1f885899c.png)





> ![](https://notes.sjtu.edu.cn/uploads/upload_669090be1545b2fa2a74777ea4d4a97b.png)
> AV1 划分实质上是 QT+MTT，MTT最大划分到两次。（少于QT+MTT，MTT depth2，故划分上AV1灵活性不如 VVC）


#### split syntax

![](https://notes.sjtu.edu.cn/uploads/upload_4c52fcd16e09eddfd9add941a695a213.png)

首先确定五种划分模式中哪些可用（复杂）
    
- example: allow quad-split 
    ![](https://notes.sjtu.edu.cn/uploads/upload_916dd4abd01b5c9051bc66232e4e5b65.png)

    

#### CST(VVC only)

- for I slice, chroma的划分可以与luma不同。
    - intra 中 chroma 通常比 luma 简单可以划分成更少的块
    - CCLM技术: 用 luma 的重建信号预测 chroma，luma 预测质量高，没必要划分成更小的块

    - ![](https://notes.sjtu.edu.cn/uploads/upload_a816b57182398d42fac421e9e609880a.png)


#### other improvements

- redundancy removal
    - ![](https://notes.sjtu.edu.cn/uploads/upload_120e85edb6970ee8553f3d4cc867ed20.png)

    - ![](https://notes.sjtu.edu.cn/uploads/upload_f7ae94d0d3fd5ba00d13625cbe952249.png)

- 图像边界强制划分
    - ![](https://notes.sjtu.edu.cn/uploads/upload_405f172caad834cfa865a71299a6babf.png)


### CU --> TU

#### ISP(a.k.a SDIP) for intra CU

- all luma CU 可以被划分为2个或4个预测模式相同的 TU
    - ![](https://notes.sjtu.edu.cn/uploads/upload_8688831be7a094f6ff08305c986aaad4.png)
- chroma 不划分，残差较小，用大的变换效率较高
- 出于硬件复杂度考虑，luma TB 必须至少包含16个像素

#### SBT for inter CU

- motivation：多数inter情况下，残差只出现在块的一侧
- CU划分为两个sub-TU：一部分为残差，另一部分强制不传

![](https://notes.sjtu.edu.cn/uploads/upload_114d1fca4e5e70adfa7d3896dd72c509.png)


### 硬件实现考虑

- VPDU 以64x64节点为单位处理，不能出现跨多个节点的单元

![](https://notes.sjtu.edu.cn/uploads/upload_1e24229de95020cec3c498ebcc1e599b.png)

- local dual tree 可能出现少于16个像素的chroma块
    - intra：luma继续划分，chroma停止划分
    - inter：不存在该问题


### 增益

![](https://notes.sjtu.edu.cn/uploads/upload_d65d11b52523ffb342ac742c2817fc76.png)





