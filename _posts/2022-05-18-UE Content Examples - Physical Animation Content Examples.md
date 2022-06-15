---
layout: post
author: chaguake
title: UE Content Examples - Physical Animation Content Examples
categories: 
- Unreal Engine
tags: 
- Animation
- UE Content Examples
---

## 一、Physical Animation Map

基于物理的动画理念是：你可以随同你的关键帧动画混入模拟结果，从而使得需要呈现"布娃娃"效果的角色产生自然模拟的感觉。

### 1.1 物理动画混合（Physics Animation Blending）

> The use of Physics Blend Weight is applied to three Actors, each is set to blend below different bones of the Skeletal Mesh (e.g. having physics applied to an animation below a specified bone in the Actor).

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.1.png"/>
<p>Physical Animation Map 1.1</p>
</div>

三个模型的`Bone Name`各不相同，在设置`Physics Blend Weight`之后，可以看到，对应骨骼及子骨骼才会呈现物理动画效果。

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.1.1.png"/>
<p>Physical Animation Map 1.1.1</p>
</div>

在动画蓝图的事件图表中使用了两个节点，以实现给角色应用物理。

*1) Set All Bodies Below Simulate Physics（设置模拟物理现象下方所有形体）*

该节点的作用是递归地激活骨架网格物体上的物理资源刚体，从给定骨骼开始模拟物理，并递归地沿着骨骼链向下移动。比如，如果你让左锁骨开始模拟，那么在骨架层次结构中位于其下面的所有骨骼也会开始模拟，最终产生一条柔软的或者是类似于布娃娃效果的手臂。简单地说，你可以把这个节点看成一个用于启动或暂停从特定节点开始模拟物理的开关。

*2) Set All Bodies Below Physics Blend Weight（设置物理混合权重下方所有形体）*

该节点简单地控制物理资源对动画骨架网格物体影响的程度。值为1.0，则使用物理驱动给定骨骼及该骨骼下的所有骨骼。值为0.0，则骨架网格物体返回到其初始关键帧动画。通常，你要在每次更新时驱动该节点，以便你可以平滑地改变Physics Blend Weight(物理混合权重)的值。

### 1.2 物理动画混合（击中反应）（Physics Animation Blending (Hit Reaction)）

> Using a Physics Blend Weight with an Impulse node to generate a Hit Reaction at the point of impact (e.g. the Actor responds to location specific damage and plays an impulse reaction while running).

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.2.png"/>
<p>Physical Animation Map 1.2</p>
</div>

运行时鼠标点击角色模型后可以看到效果。

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.2.1.png"/>
<p>Physical Animation Map 1.2.1</p>
</div>

`HitReaction`自定义事件做了两件事：计算出`Blend Weight`变量与`Is Hit`变量；给角色添加一个模拟冲量，让点击的骨骼偏离原来位置。

该事件在`PlayerCharacter`蓝图中被触发（调用）。

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.2.2.png"/>
<p>Physical Animation Map 1.2.2</p>
</div>

然后在每帧调用计算物理动画。

### 1.3 约束配置（Constraint Profiles）

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.3.png"/>
<p>Physical Animation Map 1.3</p>
</div>

在靠近台柱触发事件之后，角色模型便触发物理动画。

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.3.1.png"/>
<p>Physical Animation Map 1.3.1</p>
</div>

使用了`Set Constraint Profile for all（设置所有项的约束描述文件）`节点，`Call Trigger Actor`触发事件中该节点使用`locked`描述文件，`Call Trigger Actor End Overlap`触发事件中该节点使用`None`描述文件。

约束配置文件在*物理资产*的*描述问价*页面添加。

### 1.4 物理动画组件（physics animation component）

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.4.png"/>
<p>Physical Animation Map 1.4</p>
</div>

滑动条值为0时，角色模型在对应骨骼（及子骨骼）出现木偶娃娃效果，当值为1时，与动画序列保持一致。

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.4.1.png"/>
<p>Physical Animation Map 1.4.1</p>
</div>

添加了一个`physics animation component`组件，然后在*开始运行*事件中设置物理动画相关。

在运行过程中，有滑动条决定物理动画组件的强度乘数。

### 1.5 物理动画运动学根（physics animation component）

<div align=center>
<img src="/enclosures/2022-05-18/Physical Animation Map 1.5.png"/>
<p>Physical Animation Map 1.5</p>
</div>

滑动条值为0时，角色模型整个骨骼都出现木偶娃娃效果，当值为1时，与动画序列保持一致。

示例1.3 + 示例1.4的结果。

## 二、参考

* [Animation Content Examples](https://docs.unrealengine.com/4.27/zh-CN/Resources/ContentExamples/Animation/)
