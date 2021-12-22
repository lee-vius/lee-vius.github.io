---
layout: post
title:  "FADNet -- 阅读笔记"
date:   2021-12-06 15:30:00 +0800
categories: cv ml
icon: /assets/img_posts/2021-12-06-blog-FADNet/icon.jpg
type: blog
---
本文全称 "FADNet: A Fast and Accurate Network for Disparity Estimation"，采用神经网络的方法进行视差计算，本文提出了一个快速且准确的视差估计深度网络。本文提出一种兼顾效率和精度的视差估计模型叫 FADNet，通过结合2D卷积和相关层操作维持了较好的计算速度，利用残差结构和多尺度特征融合降低了训练的难度以提升模型精度，用更少的计算资源获得数十倍的速度提升。

<br>

# Introduction
目前基于深度学习的视差估计方法主要分为两类：一种基于2D卷积的编码器解码器架构，虽然运行速度较快但精度有待提高；第二种是基于代价空间和三维卷积的方法，可以提升视差估计的精度，但消耗的计算资源和时间非常大。

本文基于 encoder-decoder 架构的二维卷积改进而来，其结构借鉴了以往的视差估计模型 DispNetC 和 DispNetS。本文引入了**残差结构**和逐点相关性卷积操作来提升模型对于特征的抽取和表达能力，随后利用了多尺度残差学习策略和加权损失的方法来对模型由粗到精的训练。

# Method
<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-12-06-blog-FADNet/fig1.jpg">
</div>
<br>

在图像下采样中使用了基础模块 Dual-Conv，其包含一个步长为1的卷积和一层步长为2的卷积使得输出特征空间尺寸减半。受残差网络启发，本文将Dual-Conv 替换为了残差模块Dual-ResBlock，以构建更深的网络得到更好的特征表达，相比原网络增加了更多的特征抽取层，但由于Resblock的影响深度网络的训练更加容易。

相关层的概念是指在左右图中寻找相关性，correlation layer 由 DispNet 提出。但本文认为这种相关性操作和逐点卷积类似会使网络失去学习更为复杂匹配模式的能力，因此本文用两个模块来构建相关层操作，一个是通常的 3*3 卷积，第二则是逐点相乘：
<div class="equation">
$c(\vec{x_1}, \vec{x_2}) = \sum(f_1(\vec{x_1}) \cdot f_2(\vec{x_2}))$
</div>
<br>

为更好的对不同层次细节学习，并降低学习难度，深度优化过程中采用多尺度残差的结构，基本观点是优化网络通过学习不同尺度残差，与初始视差估计网络预测结果叠加来得到视差图，而不是直接输出最终的视差结果。


<br>


