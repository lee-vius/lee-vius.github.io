---
layout: post
title:  "Raft Stereo -- 论文解读"
date:   2021-12-02 16:10:00 +0800
categories: cv ml
icon: /assets/img_posts/2021-12-02-blog-raftStereo/icon.jpg
type: blog
---
本文全称 "RAFT-Stereo: Multilevel Recurrent Field Transforms for Stereo Matching"，是基于RAFT算法的又一篇立体匹配算法，在光流和立体匹配任务有着广泛的应用。相比于原生的RAFT算法，本文更关注 $X$ 轴方向的视差信息，即考虑输入的匹配图像是经过极线校正的，在网络的迭代优化过程中，使用不同尺度的特征图来增大网络的感受野，增加对大面积弱纹理区域的适应性。本文的立体匹配算法在现有的一些数据集上都取得了较好的效果，截止2021年11月，在Middlebury数据集的bad2.0指标上达到top级别。

<br>

# Introduction
本方法由 Priceton University 提出，基于Raft网络的立体匹配工作，主要是修改了RAFT 网络来解决双目立体匹配问题，基本思想是 RAFT 构建的correlation volume 和 convGRU 迭代优化过程。

算法整体流程为：给定一组图像对，双目立体匹配的目的是估计视差场，本文网络结构与RAFT类似，由三部分组成 -- 特征提取，相关性金字塔，GRU更新模块。
<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-12-02-blog-raftStereo/fig1.jpg">
</div>
<br>


# Method
**特征提取**

左右目图像分别送入 feature encoder 来提取稠密的 feature map，这些 feature map 会被用来构建相关性代价 correlation volume。feature encoder 网络由残差模块和下采样层组成，最终生成的输入图像分辨率 1/4 或 1/8 的feature map，在feature encoder 中使用instance normalization。

context encoder 网络结构和 feature encoder 一致，将 instance normalization 替换成了 batch normalization，只有左目图像会介入context encoder，生成context feature map 来初始化GRU模块的隐藏状态。
<br>

**相关性金字塔Correlation Pyramid**

相关性代价体的构建：类似于 RAFT 中构建4D correlation volume，本文在双目立体匹配中通过计算两张图像feature map的点积来构建3D correlation volume。

<div class="equation">
$C_{ijk} = \sum_h f_{ijh}\cdot g_{ikh}, C\in R^{H\times W\times W}$
</div>
<br>

代价体的理解在多篇文章中都有提到，可以看成两个特征点的特征向量之间的匹配代价，那为什么叫做匹配体 -- 相当于图1中的一个feature会和多个图2中的feature进行比较（比如水平方向交叉 $x$ 个像素）。

Correlation Pyramid 构建：和RAFT类似，本文也通过池化层把correlation volume的最后一个维度下采样实现包含不同尺度的 correlation pyramid。因而每个层级的 correlation volume 有不同的感受野，但原图分辨率没有变。

Correlation lookup：因为相关体的感受变了，因此需要一个对应的查找算子。给定当前估计的视差，我们可以在correlation volume 中反向寻找对应像素位置处的代价元素，在每个层级的correlation volume中构建1D的网络来限定查找范围，然后将不同层级的1D网络元素拼接形成一个单独的feature map。
<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-12-02-blog-raftStereo/fig2.jpg">
</div>
<br>

**多层级更新模块 Multi-Level Update Operator**
在迭代过程中，我们由初始视差会得到一系列的视差图，每次迭代会产生新的视差估计，根据视差估计值去查找correlation volume会得到correlation features，然后将该feature输入到2个卷积层，当前视差估计也会输入到2个卷积层，和context features一起送入GRU。GRU更新隐藏状态，然后我们根据隐藏状态来估计新的视差。
<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-12-02-blog-raftStereo/fig3.jpg">
</div>
<br>

RAFT更新是运行在一个固定的高分辨率尺度上，导致GRU更新过程中卷积核的感受野增长不明显，尤其是对于无纹理区域。因此，本文提出了 multi-resolution update operator。该模块可以同步的更新分辨率为1/8,1/16,1/32的feature map，多个GRU单元之间交叉互联，交错使用隐藏状态，但是查找相关体和更新最终视差始终在最高分辨率那层的GRU单元。

slow-fast GRU：由于特征图大小为1/8处的GRU更新模块需要4倍于1/16特征图的更新模块FLOPs，因而本文采用一种slow-fast的更新策略，高频率更新1/16和1/32特征图的GRUs，低频率更新1/8处的GRUs，在实际测试时能够大幅缩短运行时间。

**监督信息 Supervision**
本文将视差估计值与真值的L1距离作为loss函数，为权重指数。
<div class="equation">
$L = \sum_{i=1}^N \gamma^{N-i}\| d_{gt} - d_i \|, \gamma = 0.9$
</div>
<br>


<br>

# Conclusion
本文在众多数据集上都取得了很好的效果，ETH-3D、KITTI、Middlebury等，该文方法已开源。
