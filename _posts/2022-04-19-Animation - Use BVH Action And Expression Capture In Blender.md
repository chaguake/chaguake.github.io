---
layout: post
title: Animation - Use BVH Action And Expression Capture In Blender
categories: 
- Graphics
tags: 
- Animation
- Blender
---

因为公司PM想搞虚拟数字人这块业务，然而团队还没组建，所以就被拉去搞了两个月动画，emo。

## BVH Action

> BVH是BioVision等设备对人体运动进行捕获后产生文件格式的文件扩展名。

BVH文件包含了骨骼关键帧的旋转位移信息。在Blender中可以直接导入使用。

一般来说，BVH骨骼与目标模型的骨骼不尽相同，Blender中有两种方法，可以将BVH动作复制到目标模型的骨骼上。

* 使用Blender骨骼约束可以实现BVH动作桥接，见[链接](https://www.bilibili.com/video/BV133411B7cc)。

* 使用Blender 第三方插件实现动作重定向，例如：Auto-Rig-Pro。

## Expression Capture

表情捕捉做了相关调研，要么使用Iphone的深度摄像机，通过Faceit插件接入Blender；要么使用第三方软件工具解算视频表情，然后导入Blender，脚本驱动数据。

刚好，国内推出了一款集表情建模、绑定、面部捕捉及驱动的全流程软件——[Avatary](https://www.avatary.com/)。

<div align=center>
<img src="https://image.facegood.cc/wiki/26d0bef9e-c342-4a1c-9aae-3c3c564315ed.png"/>
<p>图一 Avatary软件</p>
</div>

使用该软件完成表情视频录制、表情数据跟踪训练以及表情数据解算。

## Mixed In Blender

我个人总结了一套开发流程：

* 第一步，使用Rigify给模型添加一套不带脸部的骨骼。

* 第二步，使用Faceit插件创建面部骨骼，并与Rigify骨骼合并。

* 第三步，使用骨骼约束将BVH动作桥接给模型Rigify骨骼。

* 第四步，使用Avatary的retargeter插件进行表情数据解算。

**演示**

<video controls height='100%' width='100%' src="/enclosures/Animation_Collage.MP4"></video>


## Others

* Blender的面部控制器不敢苟同，但自己技术又菜，不会自己撸一套面部控制器。还是使用Maya会好些，毕竟Avatary更支持对接Maya。


