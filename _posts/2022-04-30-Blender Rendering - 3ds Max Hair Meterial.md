---
layout: post
author: chaguake
title: Blender Cycles Rendering - 3ds Max Hair Model
categories: 
- Blender
tags: 
- Blender Rendering
---

## 3ds Max模型

从某模型网下载了一个3ds Max 2015的模型，头发渲染效果如下：

<div align=center>
<img src="/enclosures/2022-04-30/3ds Max 2015 rendering.png"/>
<p>3ds Max 2015 rendering</p>
</div>

贴图只有Color和Alpha：

<div align=center>
<img src="/enclosures/2022-04-30/hair texture.png"/>
<p>hair texture</p>
</div>

头发着色器：

<div align=center>
<img src="/enclosures/2022-04-30/3ds Max 2015 hair shader.png"/>
<p>3ds Max 2015 rendering</p>
</div>

## Blender Cycles渲染

新建一个原理化BSDF，连上Color和Alpha贴图。头发材质的混合模式改成`Alpha 混合`。Cycles渲染器的*光程*选项选择预设`Full Global Illumination`。

<div align=center>
<img src="/enclosures/2022-04-30/blender render 01.png"/>
<p>blender render 01</p>
</div>

可以看到在发梢处的过渡不是很自然。

<div align=center>
<img src="/enclosures/2022-04-30/blender shader 01.png"/>
<p>blender shader 01</p>
</div>

添加一个透明BSDF，与原理化BSDF混合。

<div align=center>
<img src="/enclosures/2022-04-30/blender render 02.png"/>
<p>blender render 02</p>
</div>

这时头发过渡就很自然了。

稍作修改，最终输出如下：

<div align=center>
<img src="/enclosures/2022-04-30/blender render 03.png"/>
<p>blender render 03</p>
</div>

