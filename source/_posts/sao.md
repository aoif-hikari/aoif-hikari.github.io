---
tags: VVC
mathjax: true 
title: Video Coding / SAO
date: 2021-10-20
---

> VVC环路滤波：自适应样点补偿滤波器

<!--MORE-->

## 概述

由于量化是在频域进行，而高频系数的量化失真往往更大，图像边缘一般都是高频部分，所有边缘部分的失真会更大，解码后边缘部分会出现上下波动，这就是振铃效应。重建值在原始值上下波动。

<img src="/img/VVC/zhenling.png"  />

将重建值进行分类，对波峰补偿一个负值，对波谷补偿一个正值。所以 SAO 的关键部分在于对重建像素进行分类。

VVC 中的 SAO与HEVC中基本一致，以CTB为单位对样本分group，分别对不同的group进行补偿。group index和offset被写入码流。包括两种补偿方式，边界补偿（Edge Offset,EO）和边带补偿（Band Offset,BO）。其中EO是根据样点值与周围样点值的关系来分类，而BO是根据样点本身的值来分类。

- 和 HEVC 中的 SAO 基本一致
- BD-rate 增益只有 0.1%
  - 如果没有 ALF 的话增益会高很多，能达到 0.8% 或 1%
  - 在主观质量上提升挺多
  - 复杂度低，可以适用于低延迟场景
- 主要减小了振铃效应
- 可以使用 EO（边界补偿）和 BO（边带补偿）
  - 对每个 CTB，当使用EO时：
    - 一个 CTB 只能使用一种 EO 类型
    - 4 种类的偏置会传输
  - 对边带补偿，整个样值范围被分成不同的边带
    - 样值被分类为某种边带
    - 对每种边带，传输偏置

## 补偿模式

RDO选出用EO,BO或merge。

### EO

- 通过RDO为该CTB选择使用一种梯度模式

<img src="/img/VVC/eo-g.png"  />

- 确定梯度模式后，计算该CTB中的当前像素c和相邻2个像素ab之间的关系，分为4类，每类分配一个offset，offset由RDO过程产生。规定1,2时，offset值必须是正数；对于3,4时，offset必须是负数。对于不属于以上四个类别的像素值不进行补偿。

<img src="/img/VVC/eo-classify.png" style="zoom: 67%;" />

### BO

根据样点值分为32个等间距的band（每一个band的范围是8），每个样点都会属于一个特定的区间。选择4个连续的band里的像素值进行补偿。需要补偿的边带起点和补偿值通过RDO选择。

<img src="/img/VVC/BO.png"  />

为什么4个连续的band？

- 一个CTU中大部分的pixel的取值应该会集中在很少的几个band中，使用连续的4个band能够覆盖大部分的pixel。
- 因为EO模式使用了4个offset值，为了不增加码率，BO复用了这4个offset值的syntax，这样不需要另外再增加syntax来专门表示。

### 参数融合（merge）

某一个CTB的SAO的参数可以直接使用相邻CTB的参数（色度亮度同时），只需指定是哪个相邻块（左方或上方）

## syntax

<img src="/img/VVC/sao-syntax.png"  />

<img src="/img/VVC/VQSAO.png"  />

## 结果

下表显示了在有/无ALF和CC-ALF的VTM-9.0中启用SAO对SDR内容的BD-rate改进。anchor为SAO禁用。

- 当ALF和CC-ALF被启用时，在四个测试条件下，启用SAO的平均YUV BD-速率增益为0.2%。

- 当ALF和CC-ALF被禁用时，SAO不仅平均实现了1.3%的YUV BD-速率增益，而且还提供了主观的好处，如图所示。

启用SAO的BD-rate改进与ALF部分重合。然而，考虑到SAO在HEVC中得到了很好的部署，而且所需的计算复杂度很低，SAO被保留在VVC中，以允许使用SAO来进一步提高客观和主观质量，特别是当ALF被禁用时。

<img src="/img/VVC/sao.png" style="zoom: 50%;" />

<img src="/img/VVC/sao2.png" style="zoom:;" />

## 代码

<img src="/img/VVC/codesao.png"  />