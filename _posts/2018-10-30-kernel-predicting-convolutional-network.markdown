---
title: KPCN
layout:            post
title:             "Denoising Monte-Carlo Renderings with Kernel Predicting Convolutional Network"
menutitle:         "Denoising with Kernel Predicting Convolutional Network"
date:              2018-11-04 22:40:00 +0100
category:          Paper Reading
cover:             /assets/KPCN/KPCN_teaser.png
published:		   true
author:            xianyao
tags:              Denoising
language:          EN
comments:          true
math:			   true
---

写这个之前的心理：如果不写，那么10月份就没有post了。。。结果还是拖过了10月。

本文是对于SIGGRAPH 2017的论文 [Kernel-Predicting Convolutional Networks for Denoising Monte Carlo Renderings](http://drz.disneyresearch.com/~jnovak/publications/KPCN/index.html) 做的中文简介。该论文提出使用Kernel-predicting convolutional network (KPCN) 对路径追踪（path tracing）算法渲染得到的图片进行去噪（denoising），即CNN输出并不是直接得到去噪后的图片，而是对每个像素位置获得一个filter，然后用这些filter对于输入图片进行逐像素的滤波，得到每个像素最终的RGB值作为输出。该方法在一个数据集上进行训练之后，可以迁移到场景差异很大的另一个数据集上获得较好的结果。

### 算法流程
![Fig](/assets/KPCN/framework.png "算法流程")

输入图片首先被分成Diffuse（漫反射）和Specular（镜面反射）两个分量，分别进行去噪。图像是HDR的，没有最大的像素值限制。这两个分量之和是原图像（diffuse+specular=color）。

同时作为网络输入的还有场景中的其他一些特征，如albedo（反射率）、normal（法向量）、depth（深度），同时对于这些color channel和feature channel，每个像素的方差（variance）也作为特征。输入数据共有18个channel：diffuse(3+1), specular(3+1), albedo(3+1), normal(3+1), depth(1+1)。

然后是预处理。Diffuse分量要除以albedo，specular分量经过对数变换（logarithmic transform），两个分量均经过normalization和gradient extraction的步骤，得到最终输入的diffuse和specular分量。Feature channels不做预处理。这些预处理的变换在得到输出图片的时候均会被还原（逆变换）。

第三步便是经过一个CNN得到输出结果。这个CNN在本文中显得反而不那么重要，也并不是什么fancy的结构。本文着重对比了两种情况，一是直接预测去噪图片（direct prediction，DPCN），二是kernel prediction（KPCN）。在kernel prediction时，输出21x21的滤波器，即CNN输出有441个channel。然后用这些滤波器对每个像素的邻域做滤波，得到输出的像素值，即以下公式：

<p style="text-align:center;"><img src="/assets/KPCN/gweighted.png" alt title="Weighted reconstruction" style="width:50%"/></p>

其中$p$是一个像素点，$c(p)$是$p$处的颜色值，$q$是$p$的邻域中的另一个像素，$w_{pq}$是CNN在$p$处输出的滤波器在$q$位置的权重。得到的$\hat{c}\_p$就是去噪之后$p$处的颜色值。

最后，训练使用的loss是预测的图片与reference（也就是ground truth）图片之间的$l_1$-loss。

看完这个流程之后，大概会有两个问题：

1. 为什么有这么多feature可供使用？一般的denoising，也就只有noisy image作输入。
2. 为什么要用kernel prediction这样复杂的流程？直接输出个图片不好吗？

### 这样的feature，我还能有一大把

这是由于我们进行去噪的对象是Monte Carlo rendering，也就是渲染得到的图片。具体而言，这些图片是通过路径追踪方法渲染得到的图片。这种方法是基于渲染方程（[rendering equation](https://en.wikipedia.org/wiki/Rendering_equation)），从摄像机开始发射一条路径（path），在场景中不断反射，寻找光源，计算沿着该路径会有多少光（[radiance](https://en.wikipedia.org/wiki/Radiance)）传到相机里。如下图（来自[scratchapixel.com](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-overview)）

<p style="text-align:center;"><img src="/assets/KPCN/ray-tracing.png" alt title="Ray tracing example" style="width:50%"/></p>

由于是渲染图片，场景是已知的，但场景信息不容易直接encode给CNN（首先维度就不一样高）。所幸，在path tracing这个算法中，我们的path会不断与场景发生交互（即不断反射），在每次交互的时候，就可以获得场景的一些信息。我们得到的albedo、normal、depth等feature，就是在这个path**第一次**反射时，反射面的一些性质。利用这些feature，CNN可以了解哪些像素点的范围内是同一个物体，哪些像素点比较贴近图片中的edge，从而更准确地做出预测，不管是直接预测颜色值，还是预测使用邻域中的哪些像素来估计当前像素的值。

注意，我们只使用了path第一次反射时的特征，而一条path会在场景中反射很多次，所以事实上我们还有很多可以使用的特征。但是使用过多的特征会出现边际收益递减，作者经过试验后使用了18维的输入。

### Kernel prediction可以加速训练

![](/assets/KPCN/fig9-dpkp.png "DPCN vs KPCN")

作者分析了KPCN和DPCN的效果差异，列在Fig 9里。结果表明KPCN的训练速度是DPCN的5-6倍，效果也较DPCN好一些。

究其原因，事实上KPCN对于每个像素，把滤波器的权值限制为非负并且加和为1（使用一个softmax层就可以做到）。这样使得每个像素的输出被限制在其邻域的像素值的凸包之内，缩小了搜索范围，提高了搜索效率。DPCN没有输出限制，于是搜索空间过大，训练不稳定且较慢。不管是用什么loss，如果输出范围过大，神经网络就很容易在ground truth两侧震荡，难以收敛。

KPCN事实上是受到Monte Carlo rendering denoising的前人工作启发，只在每个像素邻域内预测一个滤波器就足够了。DPCN所具有的自由度，对于去噪这一个特定任务，而且是十分需要细节的任务，并不占优势。

### 一点感慨

* 能够得到好结果，数据预处理十分重要。对于Diffuse和Specular进行分别的预处理，对于这两个分量的去噪都有决定性的意义。参见Figs. 10-13.
* 不同的数据，需要使用不同的方法，尽管task是一样的。
* Disney有那么多电影的渲染数据可以用真是好啊，别家也没条件做出这种东西来吧233。
* Rendering也被CNN入侵啦！快跑啦！
