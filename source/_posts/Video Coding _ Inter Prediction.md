---
tags: video coding
mathjax: true 
title: Video Coding / Inter Prediction
date: 2021-12-27
---

> Inter Prediction

<!--more-->

## H.265


inter prediction 以 inter PU 为单位进行。
> inter PU 划分方式：
> 
> ![](https://notes.sjtu.edu.cn/uploads/upload_1fa7e682dc597fa5200b19a48bc54df8.png)

### 运动估计

> 为当前帧的每个 PU 在已编码帧（参考图像）中搜索一个最优匹配块，计算运动向量（MV）

- MV的表示：当前块左上角坐标和参考块左上角坐标之差

- 哪些帧可以作为参考帧？
    - 从参考列表中搜


- 如何理解匹配？



- 如何搜索MV？
    - 全搜索：搜到的一定最优，复杂度高
    - TZSearch：不是标准的一部分，在HM种作为快速搜索算法。
    
        - 确定搜索起点：在以下几个位置选择RD-cost最小的作为搜索起点：median prediction MV，左/上/右上块的MV,（0，0）
        - 搜索：用棱形模板或正方形模板搜索，下图中 0 的位置为搜索起点，向外扩散，步长以2的次幂增加，选择RDcost最小的点。步长=1：4点搜索；步长<=8: 8点搜索；步长 > 8: 16点搜索
        - ![](https://notes.sjtu.edu.cn/uploads/upload_288f7fb2a26fb379982864d18707dff7.png)
        - 补充搜索：避免陷入局部最优。
            - 若步骤2选出的最优点对应的步长为1，则需要在该点周围进行二点搜索，目的是补充搜索最优点周围尚未搜索的点；
            - ![](https://notes.sjtu.edu.cn/uploads/upload_226c34f2925cfd00b8327e181678d252.png)
            - 若步骤2选出的最优点对应的步长大于某个阈值（HM默认5），则以该最优点为中心，在一定范围内做全搜索（搜索该范围内的所有点），选择率失真代价最小的作为最优点；
            - 以步骤4得到的最优点为起始点，重复步骤2-4，细化搜索。当相邻两次细化搜索得到的最优点一致时停止细化搜索。此时得到的MV即为最终MV。
        - ![](https://notes.sjtu.edu.cn/uploads/upload_acc60a5f1fd71722db46a59aee9f21fa.png)


- 提高MV精度：非整数像素
    - HEVC支持1/4亮度亚像素插值
        - ![](https://notes.sjtu.edu.cn/uploads/upload_e4d1cb423fb5f0ab6afa302189755eb4.png)
        - A为整像素处，b、h为1/2像素处，a -> 1/4, c-> 3/4
        - a -> 7抽头滤波器，用同一行的整数位置7个相邻像素值
        - $$
            \begin{array}{l}
            \mathrm{a}_{0,0}=\left(-\mathrm{A}_{-3,0}+4 * \mathrm{~A}_{-2,0}-10 * \mathrm{~A}_{-1,0}+58 * \mathrm{~A}_{0,0}+17 * \mathrm{~A}_{1,0}-5 * \mathrm{~A}_{2,0}+\mathrm{A}_{3,0}\right)>>\operatorname{shift} 1 \\
            \mathrm{~b}_{0,0}=\left(-\mathrm{A}_{-3,0}+4 * \mathrm{~A}_{-2,0}-11 * \mathrm{~A}_{-1,0}+40 * \mathrm{~A}_{0,0}+40 * \mathrm{~A}_{1,0}-11 * \mathrm{~A}_{2,0}+4 * \mathrm{~A}_{3,0}-\mathrm{A}_{4,0}\right)>>\operatorname{shift1} \\
            \mathrm{c}_{0,0}=\left(\mathrm{A}_{-2,0}-5 * \mathrm{~A}_{-1,0}+17 * \mathrm{~A}_{0,0}+58 * \mathrm{~A}_{1,0}-10 * \mathrm{~A}_{2,0}+4 * \mathrm{~A}_{3,0}-\mathrm{A}_{4,0}\right)>>\operatorname{shiftl} \\
            \mathrm{d}_{0,0}=\left(-\mathrm{A}_{0,-3}+4 * \mathrm{~A}_{0,-2}-10 * \mathrm{~A}_{0,-1}+58 * \mathrm{~A}_{0,0}+17 * \mathrm{~A}_{0,1}-5 * \mathrm{~A}_{0,2}+\mathrm{A}_{0,3}\right)>>\operatorname{shiftl} \\
            \mathrm{h}_{0,0}=\left(-\mathrm{A}_{0,-3}+4 * \mathrm{~A}_{0,-2}-11 * \mathrm{~A}_{0,-1}+40 * \mathrm{~A}_{0,0}+40 * \mathrm{~A}_{0,1}-11 * \mathrm{~A}_{0,2}+4 * \mathrm{~A}_{0,3}-\mathrm{A}_{0,4}\right)>>\operatorname{shiftl} \\
            \mathrm{n}_{0,0}=\left(\mathrm{A}_{0,-2}-5 * \mathrm{~A}_{0,-1}+17 * \mathrm{~A}_{0,0}+58 * \mathrm{~A}_{0,1}-10 * \mathrm{~A}_{0,2}+4 * \mathrm{~A}_{0,3}-\mathrm{A}_{0,4}\right)>>\operatorname{shiftl} \\
            \qquad shift1 = \min \left(4, \text { BitDepth }_{L}-8\right)
            \end{array}
            $$
        - $$
            \begin{array}{l}
            \mathrm{e}_{0,0}=\left(-\mathrm{a}_{0,-3}+4 * \mathrm{a}_{0,-2}-10 * \mathrm{a}_{0,-1}+58 * \mathrm{a}_{0,0}+17 * \mathrm{a}_{0,1}-5 * \mathrm{a}_{0,2}+\mathrm{a}_{0,3}\right)>>\operatorname{shift} 2 \\
            \mathrm{i}_{0,0}=\left(-\mathrm{a}_{0,-3}+4 * \mathrm{a}_{0,-2}-11 * \mathrm{a}_{0,-1}+40 * \mathrm{a}_{0,0}+40 * \mathrm{a}_{0,1}-11 * \mathrm{a}_{0,2}+4 * \mathrm{a}_{0,3}-\mathrm{a}_{0,4}\right)>>\operatorname{shift} 2 \\
            \mathrm{p}_{0,0}=\left(\mathrm{a}_{0,-2}-5 * \mathrm{a}_{0,-1}+17 * \mathrm{a}_{0,0}+58 * \mathrm{a}_{0,1}-10 * \mathrm{a}_{0,2}+4 * \mathrm{a}_{0,3}-\mathrm{a}_{0,4}\right)>>\operatorname{shift} 2 \\
            \mathrm{f}_{0,0}=\left(-\mathrm{b}_{0,-3}+4 * \mathrm{~b}_{0,-2}-10 * \mathrm{~b}_{0,-1}+58 * \mathrm{~b}_{0,0}+17 * \mathrm{~b}_{0,1}-5 * \mathrm{~b}_{0,2}+\mathrm{b}_{0,3}\right)>>\operatorname{shift} 2 \\
            \mathrm{j}_{0,0}=\left(-\mathrm{b}_{0,-3}+4 * \mathrm{~b}_{0,-2}-11 * \mathrm{~b}_{0,-1}+40 * \mathrm{~b}_{0,0}+40 * \mathrm{~b}_{0,1}-11 * \mathrm{~b}_{0,2}+4 * \mathrm{~b}_{0,3}-\mathrm{b}_{0,4}\right)>>\operatorname{shift} 2 \\
            \mathrm{q}_{0,0}=\left(\mathrm{b}_{0,-2}-5 * \mathrm{~b}_{0,-1}+17 * \mathrm{~b}_{0,0}+58 * \mathrm{~b}_{0,1}-10 * \mathrm{~b}_{0,2}+4 * \mathrm{~b}_{0,3}-\mathrm{b}_{0,4}\right)>>\operatorname{shift} 2 \\
            \mathrm{~g}_{0,0}=\left(-\mathrm{c}_{0,-3}+4 * \mathrm{c}_{0,-2}-10 * \mathrm{c}_{0,-1}+58 * \mathrm{c}_{0,0}+17 * \mathrm{c}_{0,1}-5 * \mathrm{c}_{0,2}+\mathrm{c}_{0,3}\right)>>\operatorname{shift} 2 \\
            \mathrm{k}_{0,0}=\left(-\mathrm{c}_{0,-3}+4 * \mathrm{c}_{0,-2}-11 * \mathrm{c}_{0,-1}+40 * \mathrm{c}_{0,0}+40 * \mathrm{c}_{0,1}-11 * \mathrm{c}_{0,2}+4 * \mathrm{c}_{0,3}-\mathrm{c}_{0,4}\right)>>\mathrm{shift} 2 \\
            \mathrm{r}_{0,0}=\left(\mathrm{c}_{0,-2}-5 * \mathrm{c}_{0,-1}+17 * \mathrm{c}_{0,0}+58 * \mathrm{c}_{0,1}-10 * \mathrm{c}_{0,2}+4 * \mathrm{c}_{0,3}-\mathrm{c}_{0,4}\right)>>\operatorname{shift} 2 \\
            \quad S h i f t 2 =\mathrm{6}
            \end{array}
            $$
    - 色度1/8亚像素插值
        - ![](https://notes.sjtu.edu.cn/uploads/upload_0b92d51dff7c5b7e2ead3cf07366117c.png)
        - $$
            \begin{array}{l}
            \mathrm{ab}_{0,0}=\left(-2 * \mathrm{~B}_{-1,0}+58 * \mathrm{~B}_{0,0}+10 * \mathrm{~B}_{1,0}-2 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{ac}_{0,0}=\left(-4 * \mathrm{~B}_{-1,0}+54 * \mathrm{~B}_{0,0}+16 * \mathrm{~B}_{1,0}-2 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{ad}_{0,0}=\left(-6 * \mathrm{~B}_{-1,0}+46 * \mathrm{~B}_{0,0}+28 * \mathrm{~B}_{1,0}-4 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{ae}_{0,0}=\left(-4 * \mathrm{~B}_{-1,0}+36 * \mathrm{~B}_{0,0}+36 * \mathrm{~B}_{1,0}-4 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{af}_{0,0}=\left(-4 * \mathrm{~B}_{-1,0}+28 * \mathrm{~B}_{0,0}+46 * \mathrm{~B}_{1,0}-6 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{ag}_{0,0}=\left(-2 * \mathrm{~B}_{-1,0}+16 * \mathrm{~B}_{0,0}+54 * \mathrm{~B}_{1,0}-4 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \mathrm{ah}_{0,0}=\left(-2 * \mathrm{~B}_{-1,0}+10 * \mathrm{~B}_{0,0}+58 * \mathrm{~B}_{1,0}-2 * \mathrm{~B}_{2,0}\right)>>\text { shift1 } \\
            \qquad \operatorname{shift} 1=\min \left(4, \text { BitDepth }_{C}-8\right)
            \end{array}
            $$


- MV 如何表示？
    - 残差MVD: MVD = MV - MVP
        - 如何预测得到MVP？高级运动向量预测AMVP(Advanced Motion Vector Prediction)模式
        - 构建候选列表，包括空域候选项和时域候选项
            - 空域候选项：按照扫描顺序，选择第一个可用的MV加入候选列表：
            - ![](https://notes.sjtu.edu.cn/uploads/upload_efedf839b9a5b45cdefa08a219e7c477.png)
            - > scaled：MV缩放
            ![](https://notes.sjtu.edu.cn/uploads/upload_7674ed50f290feed171e739517d62dcd.png)
            - 如果空域候选项合并，不足2个候选MV，需补充时域候选项
            - ![](https://notes.sjtu.edu.cn/uploads/upload_6850426fe9938dcd9b6a6f9f81543180.png)
            - 选择同位块C1的MV，C1不可用选C0
            - 根据情况MV也需要缩放
            - ![](https://notes.sjtu.edu.cn/uploads/upload_d02cd040884b4fa3d1f47ce0e56861b8.png)
        - 从候选列表中的2个选择一个作为MVP
        - 码流需要传输：残差，MVD, MVP idx
    - 把候选集里的MV作为当前的MV，直接传索引：Merge模式
        - ![](https://notes.sjtu.edu.cn/uploads/upload_8f5ab8defe0044138fc1ba029fee9b50.png)
        - 时域最多提供一个候选项，根据情况缩放，同上
        - ![](https://notes.sjtu.edu.cn/uploads/upload_6b5c50c6b68cc1b090823fdbd05c1a35.png)
        - a: 对PU2, A1无效，否则PU1 PU2相同MV，相当于没有划分PU
        - b: 对PU2, B1无效
        - 从候选集选MV作为当前的MV，直接传索引
        - ![](https://notes.sjtu.edu.cn/uploads/upload_98f487d6413abb2637256d9bd8ed72de.png)

### 预测

- 运动估计搜到的块直接作为预测块，与当前块相减得到残差
- 加权预测：通过权值因子w和偏移值offset修正预测像素
    - 默认加权预测：不用传权值
    - ![](https://notes.sjtu.edu.cn/uploads/upload_f4b2099173951db68527b8c7f8f85eae.png)
    - 显示加权预测：需要传权值
    - ![](https://notes.sjtu.edu.cn/uploads/upload_9aef6d3a5f633d8470a32345e268a416.png)


## H.266

> 运动估计：VVC中采用全搜索和TZSearch搜索算法。

### AMVP

AMVP的候选列表长度为2，候选MV总共有4种类型：

- 基于空间相邻块的空域MVP（最多允许两个）
- 基于时域同位块的时域MVP（和Merge模式时域MV的推导相同）
- 基于历史信息构建的HMVP
- 零MV

相比HEVC多了candidate：HMVP


#### Symmetric MVD coding，SMVD

对于包含双向运动信息的AMVP模式，可以使用SMVD编码模式。

使用条件：
- AMVP候选包含双向运动信息
- 当前CU的前向参考帧列表List0中距离最近的参考图像和后向参考帧列表List1中距离最近的参考图像正好处于当前图像的两侧

具体的，在编码时，仅需要编码前向MVD0，后向的MVD1可由MVD1=（-MVD0）推导得到。

![](https://notes.sjtu.edu.cn/uploads/upload_3af769cdf454d8d5be9c3ce36c8bc259.png)

### Merge

- Extended merge prediction: VVC在HEVC的基础上，扩展了Merge模式构造Merge List的方法，Merge List最多可以包含6个候选MV，构造方法如下：
    - 同HEVC: 空域MVP（最多提供4个候选MV）
    - 同HEVC: 时域MVP（最多提供1个候选MV）
    - 基于历史信息的HMVP（不限个数，直到填充到Merge List包含5个候选MV）
        - HMVP候选项来自于一个先进先出的表，表的长度为6，这个表通过已经编码块的运动信息构建，每到新的一行CTU时，这个表就要重置，即清空操作。
        - 每遇到一个帧间编码的CU（非子块）时，它相关的运动信息就会加到这个表的最后一项成为一个新的HMVP候选项。
        - 每当插入一个新的候选项时，首先要进行冗余性检查即检查待插入的项的运动信息和表中已有项的运动信息是否相同，如果不相同，则按照先进先出的规则进行插入操作；如果相同，则将相同的HMVP从维护的FIFO（先进先出）表中移除，其后所有的项都向前移动一位，将待插入的候选项插入到HMVP的表的末端。
        - 在使用HMVP候选项构建Merge候选列表时，按顺序检查HMVP列表中最新的HMVP候选（从后向前检查），在和Merge列表中现存的候选检查完冗余之后将不重复的HMVP候选添加到Merge列表中去。直到候选列表的长度达到5个。
        - 为了减少冗余性检查操作，进行了以下的简化操作：
        - 用于构建Merge list的HMVP候选项数量设为 （N<=4)?M:(8-N)，其中N表示Merge list中已有项的数目，M表示HMVP表中候选项数目。一旦Merge list中候选项的数目达到了最大允许候选项数目减一（即6-1=5），则停止从HMVP生成merge候选项的过程

    - 成对平均MVP（由Merge List中前两个候选MV平均得到）
        - 使用Merge候选列表中前两个候选项对来生成成对平均候选。Merge列表中的第一项和第二项分别定义为p0Cand和p1Cand。对于每个参考帧列表，分别根据p0Cand和p1Cand的MV是否可用来计算平均MV。即：
        - 如果两个MV在一个列表中可用，则即使这两个MV指向不同的参考帧，也对它们进行平均，并且其参考帧被设置为p0Cand对应的参考帧；
        - 如果只有一个MV可用，直接使用那一个；
        - 如果没有可用的运动矢量，则保持此列表无效。
        - 如果p0Cand和p1Cand的半像素插值滤波器指数不同，则将其滤波器指数设置为0。

    - 零MV
        - 如果计算完平均候选项后Merge还未填满，则用0MV填充候选列表，直到候选列表达到最大长度。

- Merge mode with MVD：VVC在常规Merge模式基础上，增加了“运动矢量差的”Merge模式。
    - 该模式是将常规Merge模式构建得到的Merge列表中的前两个候选MV作为初始MV，在上下左右四个方向上，进行8种步长的搜索，即在初始MV基础上加上一定的偏移MVD得到2x4x8=64个MV，并从中选出RD-Cost最小的MV作为最终的细化MV。
    - 在编码端，将初始MV在Merge List中的索引、搜索方向索引和搜素步长索引传给解码端。

- Skip：Skip模式是一种特殊的Merge模式。
    - 若当前CU被标识为Skip模式，则编码器只需要传送选中的Merge候选索引，而不需要传输变换量化后的残差。相应的在解码器只需要解析出该索引下的运动信息，然后通过运动补偿得到的预测像素值直接作为重建像素值。

- Non regular merge with CIIP
- Non regular merge with GPM
- Sub block merge list based TMVP (SbTMVP)


### Affine

#### Affine基础知识

- 仿射变换（Affine Transformation）是另外两种简单变换的叠加：线性变换 R，平移变换 T

$$
\left[\begin{array}{l}
x^{\prime} \\
y^{\prime} \\
1
\end{array}\right]=\left[\begin{array}{ccc}
R_{00} & R_{01} & T_{x} \\
R_{10} & R_{11} & T_{y} \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{l}
x \\
y \\
1
\end{array}\right]
$$

- 仿射变换变化包括：
    - 平移(translation)、缩放（Scale）
    - ![](https://notes.sjtu.edu.cn/uploads/upload_8bda874759b95f9e70ab5012e75c1ca1.png)
    - 旋转(rotate)、shear mapping
    - ![](https://notes.sjtu.edu.cn/uploads/upload_119aacd43276edef7f0ce05aff892988.png)
    - 反射（reflection）
    - ![](https://notes.sjtu.edu.cn/uploads/upload_715adf0217a1225716d480b19c032397.png)

- 仿射变换中集合中的一些性质保持不变：
（1）凸性
（2）共线性：若几个点变换前在一条线上，则仿射变换后仍然在一条线上
（3）平行性：若两条线变换前平行，则变换后仍然平行
（4）共线比例不变性：变换前一条线上两条线段的比例，在变换后比例不变

#### VVC中引入了基于块的仿射运动补偿技术

之前的帧间预测的运动估计仅考虑了简单的平移运动，无法有效表示缩放等。

VVC中引入了基于块的放射运动补偿技术，利用块中的两个控制点（四参数模型）或者三个控制点（六参数模型）的MV，来推导出整个块中每个4x4子块的MV，之后分别根据每个子块的MV通过运动补偿得到每个子块的预测值。

基于块的仿射运动补偿方式如下：
1.首先将块划分为4x4的亮度子块。
2.对每个亮度子块按下式，由控制点的MV(CPMV)计算其中心像素$(x, y)$的MV，然后四舍五入到1/16精度。


- 四参数模型

$$
\left\{\begin{array}{l}
m v_{x}=\frac{m v_{1 x}-m v_{0 x}}{W} x+\frac{m v_{1 y}-m v_{0 y}}{W} y+m v_{0 x} \\
m v_{y}=\frac{m v_{1 y}-m v_{0 y}}{W} x+\frac{m v_{1 y}-m v_{0 x}}{W} y+m v_{0 y}
\end{array}\right.
$$

- 六参数模型

$$
\left\{\begin{array}{l}
m v_{x}=\frac{m v_{1 x}-m v_{0 x}}{W} x+\frac{m v_{2 x}-m v_{0 x}}{W} y+m v_{0 x} \\
m v_{y}=\frac{m v_{1 y}-m v_{0 y}}{W} x+\frac{m v_{2 y}-m v_{0 y}}{H} y+m v_{0 y}
\end{array}\right.
$$

$（mv_{0x}，mv_{0y}）$ 是左上角控制点的MV，$（mv_{1x}，mv_{1y}）$是右上角控制点的MV，$（mv_{2x}，mv_{2y}）$是左下角控制点的MV。这些MV根据已编码的块的运动信息，用RDO选出来的

![](https://notes.sjtu.edu.cn/uploads/upload_8a7f3a6bf699f65854c43c2ae4da036c.png)

3.每个子块计算出运动向量后（如下图），根据运动向量进行运动补偿插值滤波得到每个子块的预测值。

![](https://notes.sjtu.edu.cn/uploads/upload_1b184a635c2ef24eac3947ba815726b0.png)

色度分量也划分为4x4的子块。 将4×4色度子块的MV计算为其相应的8x8亮度区域中左上和右下亮度子块的MV的平均值。

和传统的帧间预测模式类似，Affine模式也分为Affine Merge模式和Affine AMVP模式。

##### Affine Merge

> Affine Merge 通过相邻块的MV得到当前块的MV；

- 对于宽度和高度都大于或等于8的CUs，可以采用AF_MERGE模式。
- 在AF_MERGE下，当前CU的CPMV（CPMV, control point motion vector）是基于空间相邻CU的运动信息生成的。
- 最多可以有五个CPMVP候选，并用一个索引指示当前CU使用的索引。
- 以下三种类型的CPVM候选者用于形成Affine Merge候选列表：
    - 继承Affine Merge候选项：继承其相邻CU的CPMV候选项
    - 构造Affine Merge候选项：使用相邻CU的平移运动的MVs构造CPMVPs候选项
    - 零MVs



##### Affine AMVP

- Affine AMVP 根据相邻块的MV预测当前MV，得MVP，再以预测的CPMV作为搜索起点进行运动估计，得到最佳的CPMV，并将二者的差值和预测CPMV在候选列表中的索引传给解码端。


- Affine AMVP模式可应用于宽度和高度均大于或等于16的CU。在比特流中发信号通知CU级别的Affine标志以指示是否使用Affine AMVP模式，然后发信号通知另一个标志以指示是4参数Affine或6参数Affine模型。

- 在Affine AMVP模式下，需要传输其预测CPMV在候选列表中的索引以及它和运动搜索得到的实际CPMV的残差。Affine AVMP候选列表大小为2，它通过依次使用以下5种CPMV候选类型生成：
    - 继承AMVP候选项：继承其相邻CU的CPMV候选项
        - 其构建方法与Affine Merge继承AMVP候选项相同。 唯一的区别是，对于AVMP候选项，相邻CU的参考帧必须和当前帧一样。 将继承的AMVP候选项插入候选列表时，不应用修剪过程。
    - 构造Affine AMVP候选项：使用相邻CU的平移运动的MVs构造CPMVPs候选项
    - 直接使用相邻CU的平移MV
    - 同位块的时域MV
    - 零MV

构造Affine候选项的产生和Affine Merge构造方法一样。 并且需要检查相邻块的参考图片索引,要选择扫描顺序中第一个帧间编码且和当前CU有相同参考图像的块。 当当前CU用4参数Affine模式编码时，mv0和mv1都可用时，才把它们加入Affine AMVP候选列表。 当当前CU用6参数Affine模式编码时，并且所有三个CPMV均可用时，才把它们加入Affine AMVP候选列表。 否则，将构造的AMVP候选项设置为不可用。

- 如果在插入类型1和类型2的AMVP候选项之后，Affine AMVP列表候选项仍小于2，则将按顺序将平移运动MV mv0, mv1 和 mv2 (第i个MV存在时,i=0,1,2)作为当前CU所有控制点的MV。

- 如果Affine AMVP列表仍小于2且开启时域MV预测，则使用同位块的时域MV作为当前所有控制点的MV。其时域MV的获取和常规Merge模式一致。




##### SbtMVP





### Adaptive motion vector resolution，AMVR

由于实际运动通常是连续的，因此整像素精度通常不能很好地表示物体的运动。

- 在HEVC中，亮度分量的运动矢量使用1/4像素精度，色度分量的运动矢量使用1/8像素精度；
- 在VVC中，进一步提高了像素精度，亮度分量的运动矢量使用1/16像素精度，色度分量的运动矢量使用1/32像素精度。

随着像素精度的增长，预测精度增长，但随之需要大量的编码比特。

综合考虑预测精度和编码比特消耗，VVC提出了自适应运动矢量精度AMVR技术，在CU级对亮度分量MVD采用不同的像素精度进行编码。

根据当前CU**帧间预测模式**的不同，亮度分量MVD编码精度有不同的选取策略：

- 常规AMVP模式：1/4亮度像素精度，1/2亮度像素精度，整数亮度像素精度或四倍亮度像素精度。
- 仿射AMVP模式：1/4亮度像素精度，整数亮度像素精度或1/16亮度像素精度。


在编码端，需要比较各个精度下的RD-Cost并选出代价最小的编码精度作为当前CU的最优精度。为了降低编码端复杂度，避免对每个MVD精度进行四次（Affine AMVP为三次）CU级的率失真代价的比较，在VVC中使用一些快速算法来跳过除了1/4精度以外某些MVD精度的率失真代价检查。



### 预测：帧间加权预测

> 在HEVC中，通过对从两个不同参考帧获得的两个预测信号求平均和/或使用两个不同运动矢量来生成双向预测信号。但实际中，同一内容随着时间的变化有可能会产生光线强弱变化或阴影等现象，导致不同帧之间背景相似度很大，但是明暗差别较大，采用简单的平均方式预测误差较大。

HEVC中使用了帧级加权预测WP技术，VVC在此基础上，引入了CU级双向加权预测技术。

- CU级双向加权预测(Bi-prediction with CU-level weight，BCW)
- 加权预测（WP）



### 解码端技术：帧间预测后处理

为了进一步提高帧间预测的准确性，VVC采用了多种帧间预测后处理技术值，以修正帧间预测的预测像素，提高预测性能。主要包含三个：

- DMVR(Decoder side motion vector refinement，DMVR)技术：用于修正常规Merge模式双向预测MV，以Merge列表中的双向MV为初始MV，在一定范围内进行镜像搜索，以获得更精确的MV。
- BDOF(Bi-directional optical flow，BDOF)技术：使用双向光流技术来修正双向预测的预测值。
- PROF(Prediction refinement with optical flow for affine mode,PROF)技术：使用光流技术来细化基于子块的Affine运动补偿预测。


参考：https://blog.csdn.net/bigdream123/article/details/119742552
