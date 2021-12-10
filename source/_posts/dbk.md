---
tags: VVC 
mathjax: true 
title: Video Coding / Deblocking Filter
date: 2021-10-20
---

> VVC环路滤波：去块滤波器

<!--more-->

## 概述

- 主要用于去除因为基于块的运动预测和变换导致的块状伪影
  - 在 VVC 中，变换尺寸最大到 64×64 
  - DBF 主要用来提升主观质量，因此 BD-rate 数值并不是很有意义。
- 在去块滤波方面，VVC 相比于 HEVC 的主要变化是：
  - 亮度和色度都有更强（更多抽头）的去块滤波器，
    - 主要原因是 VVC 有大到 64×64 的变换块，CTU 尺寸是 128×128
  - 亮度 4×4 网格去块滤波
  - 子块边界去块滤波（仿射和 STMVP）
    - 运动矢量不同，也会有一点块效应
  - 去块决策适用于运动中更小的差异
    - HEVC 中亮度整像素，VVC 中半像素
  - 亮度自适应去块滤波
  - Tc 表扩展到 10 bit 视频
    - HEVC 中只对 8 bit 视频

下图是一个典型的块效应，在靠近块边界的地方存在一个很大的阶越，图中的 p0 和 q0 之间，这会使主观质量下降。

<img src="/img/VVC/blk.png"  />

去方块滤波的基本原理是对平滑区域的不连续边界做强滤波，对纹理丰富区域不滤波或弱滤波。处理顺序是先对整帧图像的垂直边界进行水平滤波，然 后对水平边界进行垂直滤波。这种架构使得多个水平滤波或垂直滤波可以并行进行，也可以逐 CTB 处理。

对于亮度，VVC 可以对 CU 边界、TU 子块边界的 4x4 块边界进行滤波，以及对 PU 子 块边界的 8x8 块边界进行滤波。对于色度可以对CU和TU进行8x8的边界滤波。

> PU 子块包括 SbTMVP 和仿射模式产生的子块， TU 子块包括SBT 和 ISP 模式产生的子块。对于采用矩形变换核的亮度块，也可以对 4x4 的块边界进行去方块滤波。

需要注意的是如果去方块滤波处理的是 8x8 的块边界，实际上是将 8x8 的块分成两部分单独处理，垂直边界以 8x4 为基本单位，水平边界以 4x8 为基本 单位。

## 滤波过程

### 1. 确定滤波长度

滤波长度$S_P$,$ S_Q$：$P\ Q$块中滤波的像素个数，取决于块大小。

用于决策滤波的样本：$p_{x,i}…q_{x,i}\ with\ x=0..N\ and\ i=0,3$，共$S_P+1$, $S_Q+1$个

滤波样本：$p_{x,i}…q_{x,i} with\ x=0..N−1\ and\ i=0..3$

<img src="/img/VVC/pq.png" style="zoom: 50%;" />

#### luma

CU/TU的$ S_P$/$S_Q$初始化为：

- CU/TU block side size >= 32, SP and SQ = 7.

- CU/TU block side size<= 4, SP and SQ = 1.  
- Otherwise, remaining uninitialized SP/SQ = 3.

If a CU uses PUs, SP/SQ of the CU/TU boundary is set as follows: 

- If  CU/TU  boundary  is  8  samples  distant  from  a  PU boundary, corresponding SP or SQ <= 5. 



SP/SQ of the PU boundary: 

- If the PU boundary is 8 samples distant from a CU/TU boundary, SP and SQ <= 2. 

- if the PU boundary is 4 samples distant from a CU/TU boundary, SP and SQ = 1. 

- Otherwise, SP and SQ = 3. 

SP  on  the  upper  side  of  a  horizontal  CTU  boundary,  is restricted to be <= 3. （减少行缓冲区）

> for CU/TU boundaries, SP+SQ, can be 7+7, 7+5, 5+7, 5+5, 7+3, 3+7, 5+3, 3+5, 3+3 or 1+1
>
> for PU boundaries  inside  a  CU， 3+3, 2+2 or 1+1 

这些滤波长度可以在随后的过滤决策步骤中进一步减少。

#### chroma

- the CU/TU block side sizes orthogonal to the block boundary are both >= 8 in chroma samples, $S_P $and$ S_Q $=3
- Otherwise, $S_P$ and $S_Q$ = 1.

The deblocking length for the upper side of a horizontal CTU boundary, SP, is restricted to be 1

>  $S_P+S_Q$, can thus be 3+3, 1+3 or 1+1.  

### 2. 根据边界两侧的编码模式和编码参数确定边界强度BS

边界强度值在一 定程度上反映了两个相邻块编码参数的一致性，相邻块采用的编码参数越一致， 其边界强度越小。

<img src="/img/VVC/bs.png" style="zoom:80%;" />

- 对于luma，BS=2 或 1 表示需要进行滤波，BS=0 表示不需要进行滤波。


- 对于色度，BS=2 或1且为大块边界表示需要进行滤波，其余情况不需要进行滤波。“大块”即边界长度大于等于32的块。
  - If both $S_P$ and$ S_Q $=1 and bS != 2,$ S_P$ and$S_Q $= 0. 
  - When$ S_Q$ = 3 and bS is non-zero, an additional decision based on spatial activity is made.  

### 3.根据空间活动性确定滤波强度

根据边界两侧像素的变化程度判断块的内容特性，确定该边界是否需要滤波处理。滤波强度由阈值$\beta$和$t_c$决定。

<img src="/img/VVC/eq.png" style="zoom: 25%;" /> $i=0，3$

#### luma

##### 滤波长度<=3，判断是否应用short-tap deblocking filter，同HEVC。

首先判断方程中的1式，即边界两遍的信号变化低于指定的阈值。

<img src="/img/VVC/eq1.png" style="zoom: 50%;" />

即

$\begin{array}{l}
\left|p_{2,0}-2 p_{1,0}+p_{0,0}\right|+\left|q_{2,0}-2 q_{1,0}+q_{0,0}\right|+\left|p_{2,3}-2 p_{1,3}+p_{0,3}\right|+\left|q_{2,3}-2 q_{1,3}+q_{0,3}\right|<\beta
\end{array}$

其次判断应用strong or normal。通过下式选择强滤波还是普通滤波，其中$i=0,3$<img src="/img/VVC/sw.png" style="zoom:67%;" />

如果(2)、(3)和(4)式都成立，则进行强滤波，否则进行普通滤波。

> (2)保证了边缘两边满足更小的局部自适应，
> (3)保证了边缘两边的信号的平稳，
> (4)保证边缘两遍的像素值跳变小于一个阈值。

##### 至少一边滤波长度>3，判断是否应用long-tap deblocking filter，为VVC新增。

为了去除平坦区域大尺寸块的方块效应，VVC 采用更长抽头的滤波器。设$S_P$>3。方程中的每一项增加以下项：

- 对$dPQ_i$：增加$\left|p_{3, i}-2 p_{4, i}+p_{5, i}\right|$

- 对$sP_i$
  - 若$S_P=5$，增加$|p_{3, i} - p_{s_p,i}|$
  - 若$S_P=7$，增加$\left|p_{4, i}-p_{5, i}-p_{6, i}+p_{7, i}\right|$

避免过平滑，相应$thr$也要修改。

- $thr1$改为$\beta$>>4

- $thr2$ 改为3$\beta$>>5

若不满足使用long-tap deblocking filter的条件，转为判断是否使用short-tap deblocking filter。

#### chroma

滤波长度大于3，判断四个条件，如果是420格式改为判断第0行和第1行的像素，否则还是1，3行。满足用强滤波，不满足滤波长度减到1判断是否使用弱滤波。

### 4. 滤波处理

#### luma

##### 更强滤波（long-tap deblocking filter），为双线性滤波器。

$i = 0\ to\ 3$,  $k = 0\ to$ $S_P−1$,$ l = 0\ to$ $S_Q−1$

$$
\begin{align*} &p\hbox{"}_{k,i}=Clip3\left(p_{k,i}-t_{CPk},p_{k,i}+t_{CPk},p\hbox{'}_{k,i}\right),\\ &q\hbox{"}_{l,i}=Clip3\left(q_{l,i}-t_{CQl},q_{l,i}+t_{CQl},q_{l,i}\right),\\ &p\hbox{'}_{k,i}=(f_{k}\cdot refM_{i}+(64-f_{k})\cdot refP_{i}+32) >> 6,\\ &q\hbox{'}_{l,i}=(g_{l}\cdot refM_{i}+(64-g_{l})\cdot refQ_{i}+32) >> 6,\\ &refP_{i}=(p_{Sp,i}+p_{Sp+1_{r}i}+1) >> 1,\tag{1}\\ &refQ_{i}=(q_{Sq,i}+q_{Sq+1,i}+1) >> 1,\\ &refM_{i}=\left(\sum_{m=1}^{6}(p_{m,i}+q_{m,i})+2\cdot(p_{0,i}+q_{0,i})+8\right) >> 4,\\ &\pmb{f}=\pmb{g}=\{59,50,41,32,23,14,5\},\\ &\pmb{t}_{\pmb{CP}}=\pmb{t}_{\pmb{CQ}}=(t_{C}\cdot\{6,5,4,3,2,1,1\}) >> 1, \end{align*}
$$

##### **强**滤波（short-tap strong deblocking filter） ，对每块的每行 3 个像素进行修正

$\delta_{0 s}=\left(p_{2}+2 p_{1}+2 p_{0}+2 q_{0}+q_{1}+4\right)>>3$

$\delta_{1 s}=\left(p_{2}+ p_{1}+p_{0}+q_{0}+2\right)>>2$

$\delta_{2 s}=\left(2 p_{3}+3 p_{2}+p_{1}+p_{0}+q_{0}+4\right)>>3 .$

$\delta$进行$clip$操作得到滤波后像素。相当于分别进行了$(1, 2, 2, 2, 1)/8$、$(1, 1, 1, 1)/4$和$(2, 3, 1, 1, 1)/8$的滤波操作。对$\delta$的$0，1，2$，$clip$的区间分别为$[p0-3tc, p0+3tc],[p1-2tc, p1+2tc],[p2-tc, p2+tc]$

##### **弱**滤波（short-tap normal deblocking filter）

通过下式判断修改几个像素值，满足式子则改变两个，否则改变一个。块越平坦也就意味着$P$和$Q$的边界越明显，即需要改变的像素也就越多

<img src="/img/VVC/weak.png" style="zoom: 67%;" />



对于$i=0,1,2,3$的每一行，进行滤波前还需要单独进行下式条件判断，只有满足，该行才能够进行滤波。避免对正常存在的块边界进行不必要的滤波，因为当$\delta$绝对的大于10倍tc时，一般不太可能是由于块效应导致的。

<img src="/img/VVC/weak2.png" style="zoom: 80%;" />



满足上式时，无论哪种情况，都需要对p0和q0进行滤波，

<img src="/img/VVC/p0q0.png" style="zoom: 80%;" />

$\delta_{0}$的clip区间为$[-tc, tc]$。



当条件(5)满足时，则需要对p1进行滤波；当条件(6)满足时，需要对q1进行滤波。对p1/q1滤波操作如下：   

<img src="/img/VVC/p1q1.png" style="zoom: 80%;" />

$\delta_{p1}$和$\delta_{q1}$的clip区间为$[-\frac{tc}{2}, \frac{tc}{2}]$,

> clip操作是将$\delta$的值限定到$[-c, c]$的区间,
>
> <img src="/img/VVC/clip.png" style="zoom: 70%;" />

#### chroma

##### 强滤波（long-tap  chroma deblocking filter）

$S_P$  and $ S_Q$ = 3

<img src="/img/VVC/chroma-strong.png" style="zoom: 40%;" />

$S_P$= 1 and $S_Q$ = 3

<img src="/img/VVC/13.png" style="zoom: 40%;" />

$clip$到$[p-tc, p+tc]$或$[q-tc, q+tc]$

##### 弱滤波（ short-tap chroma deblocking filter）

both $S_P$ and $S_Q$ = 1

<img src="/img/VVC/chroma-weak.png" style="zoom: 67%;" />

clip到$[-tc, tc]$后分别加/减到p0/q0上。

## luma adaptive deblocking filter

- 基本思想：块失真在更亮的区域更明显
  - 特别为 HDR 格式调优
  - 通过增加tc和 $\beta$ 获取的 qpOffset 增加了去块强度

在HEVC中滤波强度由$\beta$和tc决定，而$\beta$和tc是由平均量化参数$qPL$生成。

在VVC中为$qPL$增加一个偏移值，偏移量由重建像素的平均亮度级$LL$决定。

<img src="/img/VVC/ll.png" style="zoom: 50%;" />

<img src="/img/VVC/mapping.png" style="zoom: 67%;" />

$LL$的范围在SPS中传输。

<img src="/img/VVC/qpl.png" style="zoom: 50%;" />

$Q_{beta} = clip3(0.63, qPL+(sh\_beta\_offset\_div2<<1))$

$Q_{tc} = clip3(0.65, qPL+ 2*(bS-1)+(sh\_tc\_offset\_div2<<1))$

<img src="/img/VVC/beta.png" style="zoom: 67%;" />

<img src="/img/VVC/beta1.png" style="zoom: 50%;" />

$t_{C}=\left\{\begin{array}{cl}\left(t_{C}^{\prime}+(1<<(9-\text { BitDepth }))>>(10-\text { BitDepth }),\right. & \text { BitDepth }<10 \\ t_{C}^{\prime} *(1<<(\text { BitDepth }-10)), & \text { BitDepth } \geq 10\end{array}\right.$

在slice级中有相应的语法元素：tc_offset_div2 和 beta_offset_div2。tc和$\beta$计算时，相应地把tc−offset−div2 和beta−offset−div2的2倍加到当前的QP上，从而调整tc和$\beta$的值，实现对滤波的自适应控制。

<img src="/img/VVC/dbk.png" style="zoom: 80%;" />

## 结果

anchor为相应的工具关闭。

<img src="/img/VVC/dbfres.png" style="zoom: 50%;" />

<img src="/img/VVC/dbfres2.png" style="zoom: 33%;" />



<img src="/img/VVC/dbkres1.png" style="zoom:;" />

<img src="/img/VVC/dbkres.png" style="zoom:;" />

## 代码

<img src="/img/VVC/codedbk.png" style="zoom:;" />

