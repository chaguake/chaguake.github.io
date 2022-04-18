---
layout: post
title: Real Time High Quality Rendering - Shadow Mapping Techniques
categories: 图形学
tags: 实时渲染
---


经典阴影算法可以分为三类：Ray Casting、Shadow Mapping以及Shadow Volume。

## Shadow Mapping

> A 2-Pass Algorithm.

### 基本原理

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_theory_spaces.png"/>
<p>图一 shadow mapping theory spaces, from learnopengl</p>
</div>

* 第一步，从光源角度渲染场景，生成阴影贴图。对于点光源，应使用透视投影；对于定向光，应使用正交投影。
* 第二步，从摄像机角度渲染场景，同时使用阴影贴图判断该像素点是否位于阴影中。

*[实现]()*

### 存在问题——Self occlusion (Shadow Acne)

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_shadows.png"/>
<p>图二 Self occlusion (Shadow Acne), from learnopengl</p>
</div>

由于阴影贴图的分辨率限制，在一个像素点内所覆盖的所有场景网格都指定一个深度值（如图二黄色斜面）。导致某些场景网格的实际深度比阴影贴图的记录深度要大（如图二黑色平面），从而误判断为阴影。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_acne_diagram.png"/>
<p>图三 shadow mapping acne diagram, from learnopengl</p>
</div>

**解决方法：Adding bias**

添加一个bias值，当深度对比的差值小于bias时，设置其不为阴影。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_acne_bias.png"/>
<p>图四 shadow mapping acne bias, from learnopengl</p>
</div>

在一定程度上解决了Self occlusion的问题，但同时引入了新问题——Detached Shadow (Peter Panning)。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_peter_panning.png"/>
<p>图五 shadow mapping peter panning, from learnopengl</p>
</div>

只能动态调整bias值的大小来降低Detached Shadow的出现。比如bias值随着光线与投射面之间的夹角动态变化。

*[实现]()*

### Second-depth shadow mapping

基本思想：在生成阴影贴图时，记录像素点的第一和第二深度值，然后使用两者的平均值记录为该像素点的深度值。

缺点：物体必须要有体积。

*[实现]()*

## Percentage Closer Filtering (PCF)

// To do

*[实现]()*

## Percentage-Closer Soft Shadows (PCSS)

// To do

*[实现]()*

## Variance Soft Shadow Mapping (VSSM)

// To do

*[实现]()*

## Moment Shadow Mapping (MSM)

// To do

*[实现]()*


# 参考

* [1] [GAMES202: 高质量实时渲染](https://sites.cs.ucsb.edu/~lingqi/teaching/games202.html)

* [2] [Shadow mapping - Wikipedia](https://en.wikipedia.org/wiki/Shadow_mapping)

* [3] [Second-Depth Shadow Mapping](https://www.cs.unc.edu/techreports/94-019.pdf)

* [4] [Percentage Closer Filtering](https://http.download.nvidia.com/developer/presentations/2005/SIGGRAPH/Percentage_Closer_Soft_Shadows.pdf)

* [5] [Randima Fernando, Percentage-Closer Soft Shadows](https://developer.download.nvidia.com/shaderlibrary/docs/shadow_PCSS.pdf)

* [6] [Yang et al., Variance Soft Shadow Mapping, PG 2010](https://jankautz.com/publications/VSSM_PG2010.pdf)

* [7] [Moment Shadow Mapping](https://momentsingraphics.de/MasterThesis.html)

* [8] [实时阴影技术（Real-time Shadows）](https://www.cnblogs.com/KillerAery/p/15201310.html#percentage-closer-soft-shadowspcss)

