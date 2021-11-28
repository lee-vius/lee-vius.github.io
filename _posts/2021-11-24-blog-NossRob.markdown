---
layout: post
title:  "Noss Rob -- 论文解读"
date:   2021-11-24 22:10:00 +0800
categories: cv
icon: /assets/img_posts/2021-11-24-blog-Noss_Rob/icon.jpg
type: blog
---
本文全称《Superpixel alpha-expansion and normal adjustment for stereo matching》，在 [Middlebury](https://vision.middlebury.edu/stereo/eval3/) 视差数据集的评测网站中被命名为 NOSS ROB。本文基于超像素分割和图割算法提出了一套连续的双目视差预测方法，作者使用了一个3D的切向平面来参数化视差，并提出了两种算法来优化 Markov Random Field (MRF)，分别是 superpixel $\alpha$-expansion 和 normal adjustment。本文方法在 Middlebury 3.0 评估中取得了很好的效果。

<br>

# Introduction


# Method


<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-24-blog-Noss_Rob/fig2.jpg">
</div>
<br>
