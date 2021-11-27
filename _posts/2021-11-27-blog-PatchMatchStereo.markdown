---
layout: post
title:  "Patch Matching Stereo -- 论文解读"
date:   2021-11-27 22:10:00 +0800
categories: cv
icon: /assets/img_posts/2021-11-27-PatchMatchStereo/icon.jpg
type: blog
---
本文全称《Superpixel alpha-expansion and normal adjustment for stereo matching》，在 [Middlebury](https://vision.middlebury.edu/stereo/eval3/) 视差数据集的评测网站中被命名为 NOSS ROB。本文基于超像素分割和图割算法提出了一套连续的双目视差预测方法，作者使用了一个3D的切向平面来参数化视差，并提出了两种算法来优化 Markov Random Field (MRF)，分别是 superpixel $\alpha$-expansion 和 normal adjustment。本文方法在 Middlebury 3.0 评估中取得了很好的效果。

<br>

# Introduction


# Method
那么具体到一个像素时，我们如何选择窗口的方向呢？可以直接枚举8个可能的方向，让数据自适应地选择一个最佳的即可。如下图，横平竖直地将像素周围的区域做一个切分，所谓的8个方向就是下图中 b~d 所示案例。不难注意窗口的大小和方向是可以灵活设定的，仅有的要求是像素点必须在窗口的角落或者边缘。

<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig2.jpg">
</div>
<br>
