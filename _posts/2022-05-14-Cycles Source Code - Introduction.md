---
layout: post
author: chaguake
title: Cycles Source Code - Introduction
categories: 
- Cycles
tags: 
- Cycles Source Code
---

## 一、设计目标

> We intend to make a render engine that is interactive and easy to use, while still supporting many production features.

Cycles渲染器并不完全物理正确，是基于物理的着色。

## 二、源码结构

**Modules**

<div align=center>
<img src="/enclosures/2022-04-25/Cycles-modules-architecture.png"/>
<p>Cycles modules architecture</p>
</div>

**Folders**

| folder | description |
| ---- | ---- |
| src/app/ | 独立会话和网络渲染服务器应用程序。 |
| src/bvh/ | 用于光线跟踪的边界体积层次结构。 |
| src/device/ | 设备抽象层。 |
| src/graph/ | 通用场景节点图。 |
| src/hydra/ | 分布式任务处理系统。 |
| src/integrator/ | 路径跟踪集成器的主机端代码，调用渲染内核。 |
| src/kernel/ | 渲染内核，可以在CPU或GPU上运行。 |
| src/kernel/closures/ | svm和osl着色器后端使用的BSDF闭包。 |
| src/kernel/device/ | 特定于CPU和GPU设备的内核代码。 |
| src/kernel/integrator/ | 路径跟踪积分器内核。 |
| src/kernel/svm/ | 着色器虚拟机，用于在CPU和GPU上执行着色器图形。 |
| src/kernel/osl/ | OSL渲染服务和着色引擎仅在CPU上运行。 |
| src/kernel/osl/shaders/ | 着色器图形中使用的OSL着色器。 |
| src/scene/ | 场景图。 |
| src/session/ | 渲染会话。 |
| src/subd/ | 细分曲面。 |
| src/test/ ||
| src/util/ ||

## 三、Cycles Standalone

Cycles渲染器已集成到 Blender 中，但也可以构建为独立的渲染器。

需要在终端上单独启动：

```
cycles [options] file.xml
```

使用`Cycles --help`来获得可用选项的概览。

使用*GUI*进行渲染时，可以按`H`获得可用快捷键的概览。可以暂停、取消或重新启动渲染，或尝试允许基本相机导航的交互模式。

## 四、参考

[1] [cycles - github](https://github.com/blender/cycles)

[2] [cycles wiki](https://wiki.blender.org/wiki/Source/Render/Cycles)

