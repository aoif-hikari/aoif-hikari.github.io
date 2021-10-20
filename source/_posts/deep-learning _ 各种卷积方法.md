---
tags: deep learning 
mathjax: true 
title: deep learning / 各种卷积方法
---

> 卷积及其变种

<!-- more -->

## 2D-Convolution

<img src="https://yinguobing.com/content/images/2018/02/conv-std.jpg">

## Group Convolution【AlexNet】

![在分组卷积中，filters被拆分为不同的组，每一个组都负责具有一定深度的传统 2D 卷积的工作](https://notes.sjtu.edu.cn/uploads/upload_b10b20c60b86b91ff70f65fef21f6a09.png)

上图表示的是被拆分为 2 个filters组的分组卷积。在每个filters组中，其深度仅为传统2D-卷积的一半$D_{in}/2$，而每个filters组都包含$D_{out}/2$个filters。第一个filters组（红色）对输入层的前半部分做卷积，第二个filters组（蓝色）对输入层的后半部分做卷积。最终，每个filters组都输出了$D_{out}/2$个通道。整体上，两个组输出的通道数为$D_{out}$。之后再将这些通道堆叠到输出层中，输出层就有了$D_{out}$个通道。


## Dilated Convolution

![Dilated Convolution with a 3 x 3 kernel and dilation rate 2](https://notes.sjtu.edu.cn/uploads/upload_d89c9f7c57d4924728b65e2803d488b9.png)



## Depthwise Separable Convolution（DSC）【MobileNet/Xception】

损失精度不多的情况下大幅度降低参数量和计算量，在参数量相同的前提下，采用DSC的神经网络层数可以做的更深。

- step1. Depthwise Convolution

![](https://notes.sjtu.edu.cn/uploads/upload_a3ec37d801ec2f319a7ff8c909d6166d.png)

- step2. Pointwise Convolution

![](https://notes.sjtu.edu.cn/uploads/upload_0567db7aabb74b5289b923643fdb8854.png)

```python
'''CLASS torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)

 - dilation: controls the spacing between the kernel points; also known as the à trous algorithm.
 - groups: controls the connections between inputs and outputs. in_channels and out_channels must both be divisible by groups. For example,
		At groups=1, all inputs are convolved to all outputs.
		At groups=2, the operation becomes equivalent to having two conv layers side by side, each seeing half the input channels, and producing half the output channels, and both subsequently concatenated.
		At groups= in_channels, each input channel is convolved with its own set of filters.
'''        
class DSC(nn.Module):
    def __init__(self, in_ch, out_ch):
        super(CSDN_Tem, self).__init__()
        self.depth_conv = nn.Conv2d(
            in_channels=in_ch,
            out_channels=in_ch,
            kernel_size=3,
            stride=1,
            padding=1,
            groups=in_ch
        )
        self.point_conv = nn.Conv2d(
            in_channels=in_ch,
            out_channels=out_ch,
            kernel_size=1,
            stride=1,
            padding=0,
            groups=1
        )

    def forward(self, input):
        out = self.depth_conv(input)
        out = self.point_conv(out)
        return out

```

## Asymmetric Convolutions（AC）

![](https://notes.sjtu.edu.cn/uploads/upload_8d0cba80232d748effba02636b749964.png)

![](https://notes.sjtu.edu.cn/uploads/upload_5b7630300458886a5397d6cb4c327fd6.png)

```python
import torch.nn as nn

class CropLayer(nn.Module):

    #   E.g., (-1, 0) means this layer should crop the first and last rows of the feature map. And (0, -1) crops the first and last columns
    def __init__(self, crop_set):
        super(CropLayer, self).__init__()
        self.rows_to_crop = - crop_set[0]
        self.cols_to_crop = - crop_set[1]
        assert self.rows_to_crop >= 0
        assert self.cols_to_crop >= 0

    def forward(self, input):
        if self.rows_to_crop == 0 and self.cols_to_crop == 0:
            return input
        elif self.rows_to_crop > 0 and self.cols_to_crop == 0:
            return input[:, :, self.rows_to_crop:-self.rows_to_crop, :]
        elif self.rows_to_crop == 0 and self.cols_to_crop > 0:
            return input[:, :, :, self.cols_to_crop:-self.cols_to_crop]
        else:
            return input[:, :, self.rows_to_crop:-self.rows_to_crop, self.cols_to_crop:-self.cols_to_crop]
            
class ACBlock(nn.Module):

    def __init__(self, in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, padding_mode='zeros', deploy=False, use_affine=True, reduce_gamma=False, use_last_bn=False, gamma_init=None ):
        super(ACBlock, self).__init__()
        self.deploy = deploy
        if deploy:
            self.fused_conv = nn.Conv2d(in_channels=in_channels, out_channels=out_channels, kernel_size=(kernel_size,kernel_size), stride=stride, padding=padding, dilation=dilation, groups=groups, bias=True, padding_mode=padding_mode)
        else:
            self.square_conv = nn.Conv2d(in_channels=in_channels, out_channels=out_channels,
                                         kernel_size=(kernel_size, kernel_size), stride=stride,
                                         padding=padding, dilation=dilation, groups=groups, bias=False,
                                         padding_mode=padding_mode)
            self.square_bn = nn.BatchNorm2d(num_features=out_channels, affine=use_affine)

            center_offset_from_origin_border = padding - kernel_size // 2
            ver_pad_or_crop = (padding, center_offset_from_origin_border)
            hor_pad_or_crop = (center_offset_from_origin_border, padding)
            if center_offset_from_origin_border >= 0:
                self.ver_conv_crop_layer = nn.Identity()
                ver_conv_padding = ver_pad_or_crop
                self.hor_conv_crop_layer = nn.Identity()
                hor_conv_padding = hor_pad_or_crop
            else:
                self.ver_conv_crop_layer = CropLayer(crop_set=ver_pad_or_crop)
                ver_conv_padding = (0, 0)
                self.hor_conv_crop_layer = CropLayer(crop_set=hor_pad_or_crop)
                hor_conv_padding = (0, 0)
            self.ver_conv = nn.Conv2d(in_channels=in_channels, out_channels=out_channels, kernel_size=(kernel_size, 1),
                                      stride=stride,
                                      padding=ver_conv_padding, dilation=dilation, groups=groups, bias=False,
                                      padding_mode=padding_mode)

            self.hor_conv = nn.Conv2d(in_channels=in_channels, out_channels=out_channels, kernel_size=(1, kernel_size),
                                      stride=stride,
                                      padding=hor_conv_padding, dilation=dilation, groups=groups, bias=False,
                                      padding_mode=padding_mode)
            self.ver_bn = nn.BatchNorm2d(num_features=out_channels, affine=use_affine)
            self.hor_bn = nn.BatchNorm2d(num_features=out_channels, affine=use_affine)

            if reduce_gamma:
                assert not use_last_bn
                self.init_gamma(1.0 / 3)

            if use_last_bn:
                assert not reduce_gamma
                self.last_bn = nn.BatchNorm2d(num_features=out_channels, affine=True)

            if gamma_init is not None:
                assert not reduce_gamma
                self.init_gamma(gamma_init)


    def init_gamma(self, gamma_value):
        init.constant_(self.square_bn.weight, gamma_value)
        init.constant_(self.ver_bn.weight, gamma_value)
        init.constant_(self.hor_bn.weight, gamma_value)
        print('init gamma of square, ver and hor as ', gamma_value)

    def single_init(self):
        init.constant_(self.square_bn.weight, 1.0)
        init.constant_(self.ver_bn.weight, 0.0)
        init.constant_(self.hor_bn.weight, 0.0)
        print('init gamma of square as 1, ver and hor as 0')

    def forward(self, input):
        if self.deploy:
            return self.fused_conv(input)
        else:
            square_outputs = self.square_conv(input)
            square_outputs = self.square_bn(square_outputs)
            vertical_outputs = self.ver_conv_crop_layer(input)
            vertical_outputs = self.ver_conv(vertical_outputs)
            vertical_outputs = self.ver_bn(vertical_outputs)
            horizontal_outputs = self.hor_conv_crop_layer(input)
             = self.hor_conv(horizontal_outputs)
            horizontal_outputs = self.hor_bn(horizontal_outputs)
            result = square_outputs + vertical_outputs + horizontal_outputs
            if hasattr(self, 'last_bn'):
                return self.last_bn(result)
            return result
```

## Transposed Convolution / Deconvolution

反卷积/转置卷积是上采样过程。

![No padding，no strides，transposed](https://notes.sjtu.edu.cn/uploads/upload_1a988c699452eb8e51899e65db49db05.png)

## 计算复杂度和参数量

- `MACs` 或 `MAdds`， 加乘数，（a*x+b as 1 MAC）
- FLOPs，floating point operations. 浮点运算数
    - FLOPs = 2MACs

JVET nnvc-ctc 脚本使用 torchsummary+ptflops(get_model_complexity_info) 计算，结果对齐 MACs/pixel（=结果/输入尺寸）。torchstat计算出的差2倍。

- param各包计算结果相同







