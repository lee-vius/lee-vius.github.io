---
layout: post
title:  "maskrcnn-benchmark -- 踩坑实况"
date:   2022-03-03 21:00:00 +0800
categories: cv ml
icon: /assets/img_posts/2022-03-03-blog-maskrcnnbenchmark/icon.jpg
type: blog
---
由于需要针对mask-rcnn进行使用学习，记录一下配置环境上遇到的各种问题及其解决方案，本文使用的GitHub仓库源自[maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark)。硬件软件环境：
* Windows10
* RTX 3060
* python 3.7

<br>

# 创建虚拟环境
这里主要根据[maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark)上的`install.md`中的指示进行操作，但由于官方使用的是Linux系统，网上许多教程也是关于Ubuntu系统的问题解答，放在windows上不一定行得通，在此进行一步步的推进搭建，并将遇到的问题进行总结。

首先创建虚拟环境，使用python3.7，理论上使用python3.6不会有差别。
```
conda create -n maskrcnn python=3.7
conda activate maskrcnn
```

# 安装依赖项
这里按照原仓库中的指示进行安装
```
conda install ipython
conda install ninja yacs cython matplotlib tqdm
```
关于opencv可以使用conda自带的opencv`conda install opencv`，或者安装opencv-contrib功能更加完善`pip install opencv-contrib-python`，截止本文撰写，若不写明版本，则默认安装opencv4.5

# 安装pytorch
这里要根据对应的机器的显卡配置相应的软件，本机器为RTX 3060，需要安装CUDA11.0以上，注意如果模型跑不起来，很有可能是cuda和显卡对不上，在此不详细说明cuda配置，网上教程很多，简单来说需要的东西有这么几个，一个显卡的驱动（现在版本基本已经到4xx多了），cuda安装（10.2或者11.x视显卡而定），cudnn（根据你的cuda决定）。这些都安装完毕后，就可以安装pytorch了，官方要求安装pytorch-nightly版本，因此要找到正确的指令，pytorch官网有，记得选择对应正确的pytorch版本。
```
conda install pytorch==1.8.0 torchvision==0.9.0 torchaudio==0.8.0 cudatoolkit=11.1 -c pytorch-nightly -c conda-forge
```

# 编译cocoapi
需要到cocodataset去搞到cocoapi的源码进行编译，[cocodaset](https://github.com/cocodataset/cocoapi.git).
```
cd cocoapi/PythonAPI
python setup.py build_ext install
```
但是这里有一个tricky的问题，对于Windows版本直接如上操作，会报错说`error D8021: 无效的数值参数/Wno-cpp`。方案一，有大佬已经找到了修改办法，地址如下[修改版cocapi](https://github.com/philferriere/cocoapi)，然后再进行如上操作即可。

如果不愿意下载上述版本，则在原仓库代码上进行修改，修改内容为`cocoapi/PythonAPI`中的`setup.py`文件，文件中的第12行进行修改
```
extra_compile_args=['-Wno-cpp', '-Wno-unused-function', 'std=c99']
修改为
extra_compile_args={'gcc': ['/Qstd=c99']}
```
然后再进行上述的安装步骤。

# 编译apex
```
cd apex
python setup.py install --cuda_ext --cpp_ext
```
这里遇到的问题会非常多，首先可能会遇到你的算力过高导致编译失败，本机的算力3060为8.6，可能需要设置环境变量下调至7.5.

# 安装maskrcnn-benchmark
下一步安装mackrcnn-benchmark的包的安装，进入到maskrcnn-benchmark的源码
```
cd maskrcnn-benchmark
python setup.py build develop
```
