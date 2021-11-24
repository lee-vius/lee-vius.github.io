---
layout: post
title:  "Noss Rob -- 论文解读"
date:   2021-11-21 22:10:00 +0800
categories: cv
icon: /assets/img_posts/2021-11-21-blog-Side_Window_Filtering/icon.jpg
type: blog
---
本文主要针对图像滤波中的边缘模糊问题作出改进，在2019年被CVPR Oral接收，本文并不是一篇深度学习的文章，[论文链接](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1905.07177.pdf)。图像处理中局部窗口的使用是很常见的，大多都是将窗口的中心设定为待处理像素的位置，而本文反其道行之，先分析了这种做法并不一定适用所有场合，并提出了自己的一套滤波窗口算法。

<br>

# Introduction
对于大多数的滤波算法，我们可以考虑一个基础的情况，当一个像素处在边缘位置的时候，我们还把滤波窗口的中心放在该位置上，就会导致边缘信息发生模糊。更直接地讲，使用这种做法时，即使用非线性各向异性加权，依然会使得图像边缘的扩散。基于这一分析，本文认为可以直接将窗口的边缘放在待处理像素的位置，因此得名 side window filtering，本文基于这一思想对现在常用的滤波算法都用SWF进行替代得到了不错的效果。

<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig1.jpg">
</div>
<br>
以上图左图（中右同理）为例，图像边缘附近相邻的两个像素a和b，尽管物理上相邻，但对于他们而言选择的半窗口应是朝不同方向的，从而避免边缘模糊，本文方法在本质上**切断**了法向的扩散，而不仅是减少扩散。

**Filtering 滤波器**
* 这里对滤波器做一个简单概念理解，即在图像的一个像素经过一个滤波算法输出在这个像素上的值。大多的滤波算法需要一个滤波核，即一个窗口，通过这个窗口采集该像素周围的图像信息最终输出一个值，其算法遵循加权平均：$I_i^{\prime} = \sum_{j\in \Omega_i}\omega_{ij}q_j$，其中 i 表示一个像素的位置，$\Omega$ 表示该像素周围的一个局部窗口，$\omega$ 和 q 则是窗口中每个像素的权重和强度值。

**半窗口滤波器**
* 回归到上图的例子里，可以从数学的角度来论证全窗口滤波器对于边缘提取势必会造成模糊的问题。我们来考察一张图中的边缘位置，我们假设使用 $g(x,y)$ 来表示处在某一位置像素的图像强度。
* 考虑该点的左右两侧的极近点：$g(x - \epsilon, y)$ 和 $g(x + \epsilon, y)$，很显然由于边缘的突变，他们的导数并不相等。但考虑同一个方向的情况：$g(x - 2\epsilon, y) \approx g(x - \epsilon, y) + \epsilon g^{\prime}(x - \epsilon, y)$。
* 因此，对于上图一个边缘点 a- 和 a+ 而言，它们的近似算法所使用的窗口区域都该来源于相应的方向。因此我们不能简单地将滤波窗口怼在该像素的中心位置，而应将窗口的边缘放置在像素上。

本文除了使用泰勒展开论证边缘点的邻近像素都应来自同侧，并提出了SWF技巧作为滤波方案，还实现了众多常见的滤波算法（基于SWF），并给出了很好的效果。本文认为SWF框架可以在现有的众多应用得到广泛普及以获得更优良的效果。

<br>

# Method
那么具体到一个像素时，我们如何选择窗口的方向呢？可以直接枚举8个可能的方向，让数据自适应地选择一个最佳的即可。如下图，横平竖直地将像素周围的区域做一个切分，所谓的8个方向就是下图中 b~d 所示案例。不难注意窗口的大小和方向是可以灵活设定的，仅有的要求是像素点必须在窗口的角落或者边缘。

<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig2.jpg">
</div>
<br>
那么在这八个窗口所获得的的输出之中该如何选择呢？算法如下图所示，实际上就是选择八个算子中输出结果和原像素差值最小的那个算子。为什么要和原像素值比较取最小呢，本文认为所谓保变就是要尽量保证边缘像素提取的信息和原信息差距最小。本文使用了取平均算法去对比了Box（怼在中心的滤波器）和 SBox（本文的滤波器）得到的结果，在图一中的三种例子里，roof 型的情况并不能做到很好的保变，但其他两种情况有较好表现。

<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig3.jpg">
</div>
<br>

# Application
可以看出本文提出的是一种滤波器使用上的方法，理论上完全可以应用在当今任何用到滤波器的算法当中，只需将全窗口滤波器替换成本文提出的 SWF 即可。本文实验了图像平滑算法，去除噪点算法，HDR应用，结构纹理，深度估计，上色等不同算法，对比了使用SWF和未使用的情况，可以看出使用SWF的效果会更好地保留边缘信息。

<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig4.jpg">
</div>
<div class="home">
  <img class="image-item" src="/assets/img_posts/2021-11-21-blog-Side_Window_Filtering/fig5.jpg">
</div>
<br>