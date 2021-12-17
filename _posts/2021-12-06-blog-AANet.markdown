---
layout: post
title:  "FADNet -- 阅读笔记"
date:   2021-12-06 23:20:00 +0800
categories: cv ml
icon: /assets/img_posts/2021-12-06-blog-FADNet/icon.jpg
type: blog
---
本文全称 "AANet: Adaptive Aggregation Network for Efficient Stereo Matching"，CVPR2020 文章，针对双目匹配任务的论文。目前最好的立体匹配模型基本都在用3D卷积，计算复杂度高且占用大量存储空间，本文目的是完全替代3D卷积。

<br>

# Introduction
目前基于深度学习的视差估计方法主要分为两类：一种基于2D卷积的编码器解码器架构，虽然运行速度较快但精度有待提高；第二种是基于代价空间和三维卷积的方法，可以提升视差估计的精度，但消耗的计算资源和时间非常大。

本文基于 encoder-decoder 架构的二维卷积改进而来，其结构借鉴了以往的视差估计模型 DispNetC 和 DispNetS。本文引入了**残差结构**和逐点相关性卷积操作来提升模型对于特征的抽取和表达能力，随后利用了多尺度残差学习策略和加权损失的方法来对模型由粗到精的训练。

# Method

