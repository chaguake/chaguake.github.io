---
layout: post
author: chaguake
title: UE Content Examples - Animation Content Examples
categories: 
- UE Content Examples
tags: 
- Animation
- UE
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

### 1.6 曲线驱动动画（变形目标）（Curve Driven Animation (Morphtarget)）

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.6.png"/>
<p>Animation Map 1.6</p>
</div>

> An animation that has a Curve Float Track added which blends between two morph targets (e.g. the result is an Actor who's nose increases in size then returns to normal).

可将*变形目标*作为*骨骼网格体*的一部分导入，也可独立导入（独立于给定的网格体）。导入文件格式为FBX，其实就是Blend Shape。

### 1.7 曲线驱动动画（骨骼缩放）（Curve Driven Animation (Bone Scale)）

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.7.png"/>
<p>Animation Map 1.7</p>
</div>

> How to modify a bone of a SkeletalMeshActor during an animation inside the AnimBlueprint with a Curve Float Track (e.g. an Actor's head blends between a small/large head).

同样使用了动画序列的曲线编辑器。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.7.1.png"/>
<p>Animation Map 1.7.1</p>
</div>

在动画蓝图中使用*变换（修改）骨骼*节点直接修改`head`骨骼属性。示例中头部缩放的数据从动画序列中获取。

### 1.8 逆向运动（IK）（Inverse Kinetics (IK)）

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.png"/>
<p>Animation Map 1.8</p>
</div>

> Setting up Inverse Kinetics (IK) inside the Blueprint of an Actor for both foot IK and hand IK (e.g. an Actor who adjusts their feet along stairs and a Character who's hands deform when hitting a wall).

左边的模型没有使用IK，右脚出现明显穿模。

中间的模型使用脚部IK，可以识别不平坦的地面。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.1.png"/>
<p>Animation Map 1.8.1</p>
</div>

角色蓝图的构造脚本设置了两个数据值，前者`Scale`变量表示角色的Z值，后者`IKTrace Distance`变量表示物体胶囊体的半高乘以前者`Scale`变量。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.2.png"/>
<p>Animation Map 1.8.2</p>
</div>

`IK Foot Trace`自定义函数接受`Trace Distance`和`Socket`两个参数，返回`IKoffset`参数。

`IK Foot Trace`函数分为两个模块。第一个模块从`Socket`参数位置向下跟踪到角色底部；第二个模块获取击中某东西之后的偏移值。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.3.png"/>
<p>Animation Map 1.8.3</p>
</div>

时间图表启动一个每帧都会触发的事件，分别对左右脚进行向下追踪。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.4.png"/>
<p>Animation Map 1.8.4</p>
</div>

在动画蓝图的事件图表中，利用Pawn蓝图中存储的IK偏移量数据，生成位置矢量，以供IK执行器稍后使用。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.5.png"/>
<p>Animation Map 1.8.5</p>
</div>

在动画蓝图的动画图表中，`JumpingJacks`节点输出的结果可以被IK替换。

右边的模型使用手部IK，在击中移动的方块时手臂会停止前伸。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.6.png"/>
<p>Animation Map 1.8.6</p>
</div>

角色蓝图的构造脚本设置了`Scale`和`Hand Radius`，以及初始化双手的IK偏移量——`IKoffset Right Hand`和`IKoffset Left Hand`。事件图表则为空事件。

动画蓝图的事件图表注册了两个事件。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.7.png"/>
<p>Animation Map 1.8.7</p>
</div>

*事件蓝图初始化动画*调用了*蒙太奇播放*。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.8.png"/>
<p>Animation Map 1.8.8</p>
</div>

*事件蓝图更新动画*分为左右手的IK位置计算。以*Right Hand IK Position*为例，检测`LowerArm`与`Hand`之间的碰撞情况，在`Hand Radius`范围内出现非自身的物体，返回`IKRight Hand Alpha`值为1.0，否则为0.0。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.8.9.png"/>
<p>Animation Map 1.8.9</p>
</div>

对比中间的模型，由于阻碍物会移动，所以*双骨骼IK*的Alpha值有0.0和1.0两个选择。

### 1.9 根运动（ Root Motion）

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.9.png"/>
<p>Animation Map 1.9</p>
</div>

> Enabling Root Motion Transformation and how it impacts collision when animating a Character (e.g. a Character without Root Motion Transformation loses their collision and walks through a box, while a Character with Root Motion Transformation enabled, keeps collision and moves around the box).

根运动（Root Motion）是基于骨架根骨骼动画的角色运动。角色的根应该在运动中保持固定，并应用到角色的胶囊体中。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.9.1.png"/>
<p>Animation Map 1.9.1</p>
</div>

左边无根运动的角色在蓝图事件图表中播放了`NoRootMotionExample`动画序列。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.9.2.png"/>
<p>Animation Map 1.9.2</p>
</div>

右边根运动的角色在蓝图事件图表中先设置了运动模式，这样根运动才能工作，然后获取动画蓝图和播放动画。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 1.9.3.png"/>
<p>Animation Map 1.9.3</p>
</div>

`ABP_RootMotion`动画蓝图没有特殊的操作，在事件图表中启动了*蒙太奇播放*，播放的动画蒙太奇为`RootMotionExample_Montage`，动画片段为`RootmotionExample`。

在`RootmotionExample`的*资产详情*页面中启动*启动根运动*选项，会发现在预览窗口中角色原地运动。



### 2.1 可操作角色动画蓝图（Playable Character Animation Blueprint）

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 2.1.png"/>
<p>Animation Map 2.1</p>
</div>

> A third person controllable Character employing an Locomotion BlendSpace and an AimOffset (e.g. a playable version of the Owen Character is provided here).

具体看[这里](https://docs.unrealengine.com/4.27/zh-CN/AnimatingObjects/SkeletalMeshAnimation/CharacterSetupOverview/)。

<div align=center>
<img src="/enclosures/2022-05-13/Animation Map 2.1.1.png"/>
<p>Animation Map 2.1.1</p>
</div>

`3rdPersonPawn`角色蓝图类包含了角色和相机。



## 二、参考

* [Animation Content Examples](https://docs.unrealengine.com/4.27/zh-CN/Resources/ContentExamples/Animation/)
