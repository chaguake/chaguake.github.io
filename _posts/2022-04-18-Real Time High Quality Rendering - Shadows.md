---
layout: post
title: Real Time High Quality Rendering - Shadow Mapping Techniques
categories: 图形学
tags: 实时渲染
---

> 实时渲染充斥着大量的近似与预计算。

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

**基本思想**

在生成阴影贴图时，记录像素点的第一和第二深度值，然后使用两者的平均值记录为该像素点的深度值。

缺点：物体必须要有体积。

*[实现]()*

## Percentage Closer Filtering (PCF)

众所周知，PCF算法用于抗锯齿。

**基本思想**

查询当前像素点周围像素点的深度值，平均得到当前像素点的最终深度值。

<div align=center>
<img src="https://developer.nvidia.com/sites/all/modules/custom/gpugems/books/GPUGems3/elementLinks/08fig02.jpg"/>
<p>图六 Soft Shadow Edges via Percentage-Closer Filtering, from gpu gems 3</p>
</div>

**实现步骤**

* 第一步：对于每个像素点，分别查询其周围（5x5）像素点的深度值，分别与当前像素点的深度值比较，得到可见度值。

* 第二步：对第一步得到的可见度值集合求和平均，得到的一个介于0到1之间的数值为当前像素点的可见度。

*[实现]()*

## Percentage-Closer Soft Shadows (PCSS)

**Observation**

对于现实世界的阴影，可以得到：距离投射物越近，阴影越硬；距离投射物越远，阴影越软。

<div align=center>
<img src="/enclosures/Observation.png"/>
<p>图七 Observation</p>
</div>

阴影的软硬程度跟光源遮挡者与阴影接收者之间的距离有关。

<div align=center>
<img src="/enclosures/PCSS.png"/>
<p>图八 From Reference [5]</p>
</div>

$$
w_{Penumbra} = \frac{(d_{Receive} - d_{Blocker}) * w_{Light}}{d_{Blocker}}
$$

**实现步骤**

* 第一步：计算阴影遮挡者的平均深度值。同样是采取范围采样的方式。

* 第二步：根据阴影遮挡者的平均深度值决定过滤器大小。

* 第三步：PCF。

*[实现]()*

## Variance Soft Shadow Mapping (VSSM or VSM)

VSSM算法实现快速计算PCSS算法第一、三步的Sampling。

在PCSS的第一步实现中，目的是为了获取某个像素附近范围内不在阴影中的像素点的平均深度值。

在PCSS的第三步实现中，目的是为了获取某个像素附近范围内的平均可见度值。

VSSM算法思想：将上面两个求值变成只需求得该像素的深度值在附近范围内像素点的“排名”。

### 一个大胆假设——切比切夫不等式

VSSM算法巧妙地使用了切比切夫不等式，只需计算该像素点附近范围内的深度平均值和方差，就可以计算出该像素点的可见度值。

$$$
P(x > t) \leqslant \frac{\sigma ^2}{\sigma ^2 + (t - \mu )^2} 
$$$

其中，$\sigma^2$ 为方差，$\mu$为均值。

**均值求得方法**

* MipMap。只能获取近似的均值，特别是当像素点的Filtering范围横跨几个Maipmap单位时，值偏差更大。

* Summed Area Tables (SAT)。SAT是一个快速求得均值的算法。需要开辟一个与输入一样大小的空间（当然，可以存储在其他通道）。

**方差求得方法**

$$$
V(X) = E(X^2) - E^2(X)
$$$

通过上面的公式，要求得方差，需要另外存储深度值平方的数值。

### 另一个大胆假设

求得均值和方差之后，还不能计算出某个像素附近范围内的平均深度值。

$$$
\frac{N_{1}}{N} z_{unocc} + \frac{N_{2}}{N} z_{occ} = z_{avg}
$$$

$z_{unocc}$与$z_{occ}$是未知的，VSSM算法中，假设所有不在阴影中的像素点深度值都等于当前像素点的深度值。

<div align=center>
<img src="/enclosures/VSSM.png"/>
<p>图九 VSSM</p>
</div>

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
