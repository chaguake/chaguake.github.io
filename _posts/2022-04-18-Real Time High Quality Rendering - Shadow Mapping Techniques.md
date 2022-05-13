---
layout: post
author: chaguake
title: Real Time High Quality Rendering - Shadow Mapping Techniques
selection: false
categories:
- Graphics
tags:
- Real-Time Rendering
---

> 实时渲染充斥着大量的近似与预计算。

经典阴影算法可以分为三类：Ray Casting、Shadow Mapping以及Shadow Volume。

## 一、Shadow Mapping

> A 2-Pass Algorithm.

### 1.1 基本原理

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_theory_spaces.png"/>
<p>图 shadow mapping theory spaces, from learnopengl</p>
</div>

* 第一步，从光源角度渲染场景，生成阴影贴图。对于点光源，应使用透视投影；对于定向光，应使用正交投影。
* 第二步，从摄像机角度渲染场景，同时使用阴影贴图判断该像素点是否位于阴影中。

#### 1.1.1 实现

以github上的[Vulkan](https://github.com/SaschaWillems/Vulkan)项目中的`examples/shadowmapping`为例。

<div align=center>
<img src="/enclosures/2022-04-18/shadow mapping In Vulkan.JPG"/>
<p>图 shadow mapping In Vulkan</p>
</div>

先看`shadowmapping.cpp`中的`buildCommandBuffers`函数，执行了两个render pass：

**第一个render pass生成shadow map。**

在`preparePipelines`函数中可以看到，`offscreen`渲染管线只需要vertex shader（offscreen.vert.spv）。

**第二个render pass使用shadow map渲染场景。**

scene.frag中`textureProj`函数查找阴影贴图时的坐标值`shadowCoord.st`是在scene.vert中生成的。

```
outShadowCoord = ( biasMat * ubo.lightSpace * ubo.model ) * vec4(inPos, 1.0);
```

需要将场景的坐标从世界坐标系转换到光源坐标系上。

### 1.2 存在问题——Self occlusion (Shadow Acne)

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_shadows.png"/>
<p>图 Self occlusion (Shadow Acne), from learnopengl</p>
</div>

由于阴影贴图的分辨率限制，在一个像素点内所覆盖的所有场景网格都指定一个深度值（如上图黄色斜面）。导致某些场景网格的实际深度比阴影贴图的记录深度要大（如上图黑色平面），从而误判断为阴影。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_acne_diagram.png"/>
<p>图 shadow mapping acne diagram, from learnopengl</p>
</div>

**解决方法：Adding bias**

添加一个bias值，当深度对比的差值小于bias时，设置其不为阴影。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_acne_bias.png"/>
<p>图 shadow mapping acne bias, from learnopengl</p>
</div>

在一定程度上解决了Self occlusion的问题，但同时引入了新问题——Detached Shadow (Peter Panning)。

<div align=center>
<img src="https://learnopengl.com/img/advanced-lighting/shadow_mapping_peter_panning.png"/>
<p>图 shadow mapping peter panning, from learnopengl</p>
</div>

只能动态调整bias值的大小来降低Detached Shadow的出现。比如bias值随着光线与投射面之间的夹角动态变化。

### 1.3 Second-depth shadow mapping

#### 1.3.1 基本思想

在生成阴影贴图时，记录像素点的第一和第二深度值，然后使用两者的平均值记录为该像素点的深度值。

缺点：物体必须要有体积。

#### 1.3.2 实现

// To do

## 二、Percentage Closer Filtering (PCF)

众所周知，PCF算法用于抗锯齿。

### 2.1 基本思想

查询当前像素点周围像素点的深度值，平均得到当前像素点的最终深度值。

<div align=center>
<img src="https://developer.nvidia.com/sites/all/modules/custom/gpugems/books/GPUGems3/elementLinks/08fig02.jpg"/>
<p>图 Soft Shadow Edges via Percentage-Closer Filtering, from gpu gems 3</p>
</div>

### 2.2 实现步骤

* 第一步：对于每个像素点，分别查询其周围（5x5）像素点的深度值，分别与当前像素点的深度值比较，得到可见度值。

* 第二步：对第一步得到的可见度值集合求和平均，得到的一个介于0到1之间的数值为当前像素点的可见度。

#### 2.2.1 实现

以github上的[Vulkan](https://github.com/SaschaWillems/Vulkan)项目中的`examples/shadowmapping`为例。

简单地Vulkan示例源码解剖见上文。

```
float filterPCF(vec4 sc)
{
	ivec2 texDim = textureSize(shadowMap, 0);
	float scale = 1.5;
	float dx = scale * 1.0 / float(texDim.x);
	float dy = scale * 1.0 / float(texDim.y);

	float shadowFactor = 0.0;
	int count = 0;
	int range = 1;
	
	for (int x = -range; x <= range; x++)
	{
		for (int y = -range; y <= range; y++)
		{
			shadowFactor += textureProj(sc, vec2(dx*x, dy*y));
			count++;
		}
	
	}
	return shadowFactor / count;
}
```

上述代码很简单，对当前像素点周围3x3范围求均值。

<div align=center>
<img src="/enclosures/2022-04-18/Shadow Mapping With PCF In Vulkan.JPG"/>
<p>图 Shadow Mapping With PCF In Vulkan</p>
</div>

* 均匀圆盘分布采样（Uniform-Disk Sample）：圆范围内随机取一系列坐标作为采样点；看上去比较杂乱无章，采样效果的 noise 比较严重。

* 泊松圆盘分布采样（Poisson-Disk Sample）：圆范围内随机取一系列坐标作为采样点，但是这些坐标还需要满足一定约束，即坐标与坐标之间至少有一定距离间隔。

<div align=center>
<img src="/enclosures/2022-04-18/disk_sample.png"/>
<p>图 disk sample</p>
</div>

可以加大范围，然后使用分布采样函数来提速。

```
float filterPCF2(vec4 sc)
{
	uniformDiskSamples(sc.xy);

	ivec2 texDim = textureSize(shadowMap, 0);
	float scale = 1.5;
	float dx = 1.0 / float(texDim.x) * scale;
	float dy = 1.0 / float(texDim.y) * scale;

	float shadowFactor = 0.0;
	  for( int i = 0; i < NUM_SAMPLES; i ++ ) {
		shadowFactor += textureProj(sc, vec2(poissonDisk[i].x * dx, poissonDisk[i].y * dy));
	  }

  float shadow = shadowFactor / float(NUM_SAMPLES);
  return shadow;
}
```

<div align=center>
<img src="/enclosures/2022-04-18/Shadow Mapping With PCF In Vulkan 2.JPG"/>
<p>图 Shadow Mapping With PCF In Vulkan</p>
</div>

## 三、Percentage-Closer Soft Shadows (PCSS)

### 3.1 Observation

对于现实世界的阴影，可以得到：距离投射物越近，阴影越硬；距离投射物越远，阴影越软。

<div align=center>
<img src="/enclosures/2022-04-18/Observation.png"/>
<p>图 Observation</p>
</div>

阴影的软硬程度跟光源遮挡者与阴影接收者之间的距离有关。

<div align=center>
<img src="/enclosures/2022-04-18/PCSS.png"/>
<p>图 From Reference [5]</p>
</div>

$$
w_{Penumbra} = \frac{(d_{Receive} - d_{Blocker}) * w_{Light}}{d_{Blocker}}
$$

### 3.2 实现步骤

* 第一步：计算阴影遮挡者的平均深度值。同样是采取范围采样的方式。

* 第二步：根据阴影遮挡者的平均深度值决定过滤器大小。

* 第三步：PCF。

#### 3.2.1 实现

// To do

## 四、Variance Soft Shadow Mapping (VSSM or VSM)

VSSM算法实现快速计算PCSS算法第一、三步的Sampling。

在PCSS的第一步实现中，目的是为了获取某个像素附近范围内不在阴影中的像素点的平均深度值。

在PCSS的第三步实现中，目的是为了获取某个像素附近范围内的平均可见度值。

VSSM算法思想：将上面两个求值变成只需求得该像素的深度值在附近范围内像素点的“排名”。

### 4.1 一个大胆假设——切比切夫不等式

VSSM算法巧妙地使用了切比切夫不等式，只需计算该像素点附近范围内的深度平均值和方差，就可以计算出该像素点的可见度值。

$$
P(x > t) \leqslant \frac{\sigma ^2}{\sigma ^2 + (t - \mu )^2} 
$$

其中，$\sigma^2$ 为方差，$\mu$为均值。

**均值求得方法**

* MipMap。只能获取近似的均值，特别是当像素点的Filtering范围横跨几个Maipmap单位时，值偏差更大。

* Summed Area Tables (SAT)。SAT是一个快速求得均值的算法。需要开辟一个与输入一样大小的空间（当然，可以存储在其他通道）。

**方差求得方法**

$$
V(X) = E(X^2) - E^2(X)
$$

通过上面的公式，要求得方差，需要另外存储深度值平方的数值。

### 4.2 另一个大胆假设

求得均值和方差之后，还不能计算出某个像素附近范围内的平均深度值。

$$
\frac{N_{1}}{N} z_{unocc} + \frac{N_{2}}{N} z_{occ} = z_{avg}
$$

$z_{unocc}$与$z_{occ}$是未知的，VSSM算法中，假设所有不在阴影中的像素点深度值都等于当前像素点的深度值。

<div align=center>
<img src="/enclosures/2022-04-18/VSSM.png"/>
<p>图 VSSM</p>
</div>

#### 4.2.1 实现

// To do

## 五、Moment Shadow Mapping (MSM)

// To do

#### 5.1.1 实现

// To do


## 六、参考

* [1] [GAMES202: 高质量实时渲染](https://sites.cs.ucsb.edu/~lingqi/teaching/games202.html)

* [2] [Shadow mapping - Wikipedia](https://en.wikipedia.org/wiki/Shadow_mapping)

* [3] [Second-Depth Shadow Mapping](https://www.cs.unc.edu/techreports/94-019.pdf)

* [4] [Percentage Closer Filtering](https://http.download.nvidia.com/developer/presentations/2005/SIGGRAPH/Percentage_Closer_Soft_Shadows.pdf)

* [5] [Randima Fernando, Percentage-Closer Soft Shadows](https://developer.download.nvidia.com/shaderlibrary/docs/shadow_PCSS.pdf)

* [6] [Yang et al., Variance Soft Shadow Mapping, PG 2010](https://jankautz.com/publications/VSSM_PG2010.pdf)

* [7] [Moment Shadow Mapping](https://momentsingraphics.de/MasterThesis.html)

* [8] [实时阴影技术（Real-time Shadows）](https://www.cnblogs.com/KillerAery/p/15201310.html#percentage-closer-soft-shadowspcss)

