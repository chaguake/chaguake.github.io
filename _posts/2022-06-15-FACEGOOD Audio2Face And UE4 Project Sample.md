---
layout: post
author: chaguake
title: FACEGOOD Audio2Face And UE4 Project Sample
categories: 
- Unreal Engine
tags: 
- Animation
---

## 一、FACEGOOD Audio2Face

FACEGOOD开源了一个语音驱动数字人口型技术，基于2017年Siggraph一篇论文。

具体实现看FACEGOOD官方github介绍，FACEGOOD官方提供一个AiSpeech测试服务和一个UE4打包程序，大致工作流程如下：UE4程序发送录音指令和接收BS权重数据，两者通过UDP通信，AiSpeech测试服务实现ASR、TTS和Voice2Face功能，其中会用到第三方平台服务功能。

<div align=center>
<img src="/enclosures/2022-06-15/pipeline.PNG"/>
<p>AiSpeech测试服务 pipeline</p>
</div>

## 二、UE4 Project Sample

一开始我直接使用FACEGOOD官方UE4打包程序中的插件来实现驱动MetaHuman模型，但是搞了很久也没看懂该插件的调用流程。干脆自己从零开始，实现UE4端的功能。

通过抓包可以得知两者间数据对接的格式。

从github上随便找了一个叫[udp-ue4-plugin-win64](https://github.com/is-centre/udp-ue4-plugin-win64)的UDP插件，简单修改下便可完成目标功能。

然后创建一个蓝图接口，将BS权重数据从UDP接收者蓝图传递到MetaHuman的Actor蓝图类中，然后再传递到Face_AnimBP。

<div align=center>
<img src="/enclosures/2022-06-15/传递BS权重数据.PNG"/>
<p>传递BS权重数据</p>
</div>

在Face_AnimBP的动画图表中，断开Live Link的实时链接姿势，改成由传递过来的BS权重数据驱动。根据FACEGOOD官方给出的映射文件找到对应的BS曲线，一一对应即可。

<div align=center>
<img src="/enclosures/2022-06-15/BS权重数据驱动.PNG"/>
<p>BS权重数据驱动</p>
</div>

大功告成，但是可以看到明显的嘴部抖动，还需优化。

<video controls height='100%' width='100%' src="/enclosures/2022-06-15/video.mp4"></video>


## 三、参考

* [FACEGOOD-Audio2Face](https://github.com/FACEGOOD/FACEGOOD-Audio2Face)

* [Audio-Driven Facial Animation by Joint End-to-End Learning of Pose and Emotion](chrome-extension://oemmndcbldboiebfnladdacbdfmadadm/https://research.nvidia.com/sites/default/files/publications/karras2017siggraph-paper_0.pdf)

* [udp-ue4-plugin-win64](https://github.com/is-centre/udp-ue4-plugin-win64)