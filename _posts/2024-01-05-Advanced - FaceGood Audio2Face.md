---
layout: post
author: chaguake
title: Advanced - FACEGOOD Audio2Face
categories: 
- Unreal Engine
tags: 
- Animation
---


# 【2.0版本】基于FaceGood Audio2Face训练模型，LiveLink驱动UE数字人


> 嗨，你好！**Codart Studio**专注于输出数字艺术与计算机技术领域的前沿干货内容。关注【**码上艺术**】，开阔新视野。

一年前在自己的Github Page写了一篇[FACEGOOD Audio2Face And UE4 Project Sample](https://www.chaguake.com/facegood-audio2face-and-ue4-project-sample)文章。


在没有SEO情况下，没想到阅读量有1000+，还是有点小开心的！感兴趣的小伙伴可以点击下面链接或原文跳转阅读。

https://www.chaguake.com/facegood-audio2face-and-ue4-project-sample

这几天花了点时间重新优化了下，效果还不错。

## 技术实现&效果优化

FaceGood官方训练代码Github地址：

https://github.com/FACEGOOD/FACEGOOD-Audio2Face

Audio2Face原理这块就不展开说了，基于英伟达2017年发表的[论文](https://research.nvidia.com/sites/default/files/publications/karras2017siggraph-paper_0.pdf)。

当时的做法直接仿照FaceGood官方，使用科大讯飞平台处理音频数据，通过UDP将BlendShape数据推送到UE端，再在蓝图中连线……想想都觉得麻烦！

### 接入EmotiVoice

音频来源做了调整，改成使用网易EmotiVoice输出的数据。

想要了解EmotiVoice的小伙伴可以看往期这篇[文章](https://mp.weixin.qq.com/s?__biz=MzIzODg5NTYxOA==&mid=2247484183&idx=1&sn=4ba95f29852f73b6afa70801a1b253a9&chksm=e933147ade449d6ceca027b0417aadd987fc96568e402791003fcf904a206b3a1c975c549994#rd)。

### 音频处理

以前看那篇论文时没细看，没发现在静默音频数据输入下也会产生不可控的BS数据输出。

<div align=center>
<img src="https://cdn.jsdelivr.net/gh/chaguake/PictureBed/2023/12/19/1702992006050-09a1c0e0-1131-48e9-82aa-e2e9286fc9f4.PNG"/>
<p>BS权重数据驱动</p>
</div>

使用`split_on_silence`函数将音频处理成单独的每句话。

### LiveLink

参考Iphone的Live Link Face APP，将BS数据封装推送给UE。

这里推荐一个Github开源库，可以快速实现LiveLink数据包封装：

https://github.com/think-biq/LLV

### 效果展示

<video controls height='100%' width='100%' src="/enclosures/2024-01-05/video.mkv"></video>


#### 推荐阅读

- [手把手教你搭建免费的个人ChatGPT网站，国内就能访问！](https://mp.weixin.qq.com/s?__biz=MzIzODg5NTYxOA==&mid=2247484166&idx=1&sn=426d8ff93ee910c2e0ea556364a4e3ad&chksm=e933146bde449d7de806c7f74c2a07deded684ba6cf49fcbbc18ccc78a8318a1e37e3c23a27b#rd)

- [网易EmotiVoice小白安装教程，视频配音不求人！](https://mp.weixin.qq.com/s?__biz=MzIzODg5NTYxOA==&mid=2247484183&idx=1&sn=4ba95f29852f73b6afa70801a1b253a9&chksm=e933147ade449d6ceca027b0417aadd987fc96568e402791003fcf904a206b3a1c975c549994#rd)

- [AI生成的音频过于单调？来试试用下Suno & Bark！](https://mp.weixin.qq.com/s?__biz=MzIzODg5NTYxOA==&mid=2247484217&idx=1&sn=18b67237dbbdad8a3999c206306c8aa9&chksm=e9331454de449d42adf3b7a946094c0d04bcbf1db987d6db9e1d2af94a4df993a0b44c5ac7c0#rd)


---

<center>
    <img src="https://cdn.jsdelivr.us/gh/chaguake/PictureBed/2023/12/03/1701588180751-4e1c6506-3a64-4783-be2d-9ecbee5a64ad.jpg" style="width: 100px;">
</center>

关注【**码上艺术**】，获取更多前沿技术干货！
