---
layout: post
author: chaguake
title: Animation - UE 4.27.2 Animation Content Examples
categories: 
- Animation
tags: 
- Animation
- UE4
---

## 一、Animation Map

### 1.1 动画 - 使用动画资产（Animation - Using Animation Asset）

> A SkeletalMeshActor that is set to play an AnimationSequence asset assigned through the Details panel of the Actor (e.g. Actor loops and performs jumping jacks).

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.1.png"/>
<p>Animation Map 1.1</p>
</div>

示例1.1直接在骨骼网格体Actor上添加动画序列。

### 1.2 动画蓝图（Animation Blueprint）

> Two examples are shown: one where a Blueprint is used to tell an Actor to play an animation, the other shows an Animation Blueprint which tells the Actor to play an animation (e.g. this example shows how to pass a variable from a Blueprint to an Animation Blueprint and demonstrates blending from an existing pose to a new animation).

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2.png"/>
<p>Animation Map 1.2</p>
</div>

示例1.2使用*播放动画函数*和*动画蓝图*来播放动画，当镜头靠近两个触发器时，将触发动画蓝图事件。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2 Left.png"/>
<p>Animation Map 1.2 Left</p>
</div>

左边的示例使用*播放动画函数*指定播放的动画序列。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2 Right 1.png"/>
<p>Animation Map 1.2 Right 1</p>
</div>

右边的示例触发事件之后，`SimpleAnimationBPCharacter`蓝图类设置了一个`PlayAnimation`属性。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2 Right 2.png"/>
<p>Animation Map 1.2 Right 2</p>
</div>

`SimpleAnimationBPCharacter`蓝图类的骨骼网格体设置了动画蓝图`ABP_SimpleAnimation`，该动画蓝图同样添加了`PlayAnimation`属性。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2 Right 3.png"/>
<p>Animation Map 1.2 Right 3</p>
</div>

在动画蓝图`ABP_SimpleAnimation`事件图表中，可以看到，当蓝图更新动画之后，会将当前蓝图的类型转换为`SimpleAnimationBPCharacter`蓝图类，然后获取`PlayAnimation`属性，并赋值给自身的`PlayAnimation`属性。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.2 Right 4.png"/>
<p>Animation Map 1.2 Right 4</p>
</div>

动画蓝图`ABP_SimpleAnimation`的动画图表的状态机比较简单，只有两个状态。两个状态的切换规则由`PlayAnimation`属性决定。

### 1.3 运动混合空间（Locomotion BlendSpace）

> How a Locomotion BlendSpace can be used to blend an Actor's movement in different directions and at different speeds (e.g. an Actor is shown blending between walking/running backwards, forwards, left, and right at different speeds).

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.3.png"/>
<p>Animation Map 1.3</p>
</div>

示例1.3使用混合空间来控制动画输出。方向和速度两个属性由两个Slider蓝图决定。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.3.2.png"/>
<p>Animation Map 1.3.2</p>
</div>

同示例1.2右，使用了动画蓝图`ABP_SimpleAnimation`，获取方向和速度两个属性的方式也是一致。

状态机使用了*混合空间播放器*组件，该组件的*细节*面板设置了混合空间为`LocomotionBlendspace`，同时传递外部控制输入的方向和速度两个属性值。双击该组件可以查看并编辑混合空间。

混合空间相关的知识查看[这里](https://docs.unrealengine.com/4.27/zh-CN/AnimatingObjects/SkeletalMeshAnimation/Blendspaces/)。

### 1.4 瞄准偏移（AimOffset）

> An AimOffset is showcased which will allow you to control and blend between different poses for aiming along Yaw or Pitch values (e.g. an Actor is shown blending between various aiming positions).

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.4.png"/>
<p>Animation Map 1.4</p>
</div>

瞄准偏移本质上来说是混合空间，不过它是用来设计瞄准武器的动画。

### 1.5 动画蒙太奇（Animation Montage）

> Playing an Animation Montage from a Blueprint as well as blending into/out of the animation is shown (e.g. an Actor blends between an aim pose and throwing punches).

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.5.png"/>
<p>Animation Map 1.5</p>
</div>

示例1.5同样采用触发方式来启动或停止*蒙太奇*。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.5.2.png"/>
<p>Animation Map 1.5.2</p>
</div>

在事件图表中，两个触发事件分别对应*蒙太奇播放*和*蒙太奇停止*，*蒙太奇播放*组件引用了动画蒙太奇`Punch_Montage`资产。

`AnimMontageBP`蓝图类使用了动画蓝图`ABP_SimpleAnimMontage`。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.5.3.png"/>
<p>Animation Map 1.5.3</p>
</div>

可以看到，动画蓝图`ABP_SimpleAnimMontage`使用了`fullbodyslot`插槽，对应的是动画蒙太奇`Punch_Montage`中的插槽。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.5.4.png"/>
<p>Animation Map 1.5.4</p>
</div>

动画蒙太奇`Punch_Montage`中分了3个蒙太奇片段，在没有退出*蒙太奇播放*时，会在`Punch1`和`Punch2`之间循环。

动画蒙太奇相关的知识查看[这里](https://docs.unrealengine.com/4.27/zh-CN/AnimatingObjects/SkeletalMeshAnimation/AnimMontage/)。


## 二、PhysicalAnimation Map


## 三、参考

* [Animation Content Examples](https://docs.unrealengine.com/4.27/zh-CN/Resources/ContentExamples/Animation/)
