---
layout: post
author: chaguake
title: UE Content Examples - Cloth Content Examples
categories: 
- UE Content Examples
tags: 
- UE
---

## 一、Cloth

### 1.1 APEX（老版本）流程和资产

<div align=center>
<img src="https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Physics/Cloth/Overview/OldWorkflow.webp"/>
<p>原有的APEX布料工作流程</p>
</div>

在老版本的流程中，需要分别制作并导入FBX和APX到UE。将APEX布料（.apx或.apb）文件导入编辑器有2种方法。

在*骨架网格体编辑器*中，你可以使用*资产细节（Asset Details）*的*布料（Clothing）*分段下的*添加APEX布料文件（Add APEX clothing file...）*按钮。

<div align=center>
<img src="https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Physics/Cloth/Overview/APEXLegacyImport.webp"/>
<p>APEXLegacyImport</p>
</div>

或者，也可以启用编辑器内的布料工具，使用*布料（Clothing）*窗口中*资产（Assets）*下的*加号（+）*按钮来导入APEX文件。

<div align=center>
<img src="https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Physics/Cloth/Overview/ImportAPEX.webp"/>
<p>ImportAPEX</p>
</div>

使用此方法时，APEX布料资产将转换到编辑器内工具，在创建不同参数（如*最大距离（Max Distance）*和*逆止（Backstop）*）遮罩的同时，相应地创建和指定至渲染网格体。

### 1.2 虚幻引擎布料工作流程

<div align=center>
<img src="https://docs.unrealengine.com/4.27/Images/InteractiveExperiences/Physics/Cloth/Overview/NewWorkflow.webp"/>
<p>虚幻引擎布料工作流程</p>
</div>

布料工具在编辑器中可用后，开发人员将能直接使用虚幻引擎来编写内容，而不需要外部依赖项。

操作：右键骨骼网格体的部位，然后创建布料资产。

## 二、Cloth Map

### 2.1 Apex布料和风向源（Apex Cloth and WindDirectionalSource）

<div align=center>
<img src="/enclosures/2022-05-20/Cloth Map 1.1.png"/>
<p>Cloth Map 1.1</p>
</div>

演示了带有Apex布料资产的骨骼网格体。

### 2.2 Apex布料属性（已导入）（Apex Cloth Properties (imported)）

<div align=center>
<img src="/enclosures/2022-05-20/Cloth Map 1.2.png"/>
<p>Cloth Map 1.2</p>
</div>

设置具有不同属性的Apex布料资产，可以模拟不同类型的布料（例如，模拟粗麻布、丝绸、橡胶和厚革材质）。

### 2.3 Apex布料碰撞（Apex Cloth Collision）

<div align=center>
<img src="/enclosures/2022-05-20/Cloth Map 1.3.png"/>
<p>Cloth Map 1.3</p>
</div>

展示了两个使用*胶囊体刚体（Capsule Rigid Bodies）*进行碰撞的示例，可以帮助避免布料与骨骼网格体中不模拟布料的部分相交。

### 2.4 逆止（BackStop）

<div align=center>
<img src="/enclosures/2022-05-20/Cloth Map 1.4.png"/>
<p>Cloth Map 1.4</p>
</div>

布料碰撞的另一种方式，其中对布料进行加权的骨骼决定了布料可移动的距离（性能表现更佳）。

布料的遮罩有几种类型，可以混合使用：

* 最大距离（Max Distance）：布料上的点从其动画位置移动的最远距离。

* 逆止距离（Backstop Distance）：布料上的点所用偏移，以限制最大距离（Max Distance）内的移动。

* 逆止半径（Backstop Radius）：与最大距离（Max Distance）相交时，可防止布料上已绘制的点进入该区域的球体半径。

* 动画驱动乘数（Anim Drive Multiplier）：用于驱动弹簧朝动画设置的位置拉伸布料，使动画师能够控制过场或动画驱动的场景。

	* 也可在运行时使用Actor之间的对象来驱动，蓝图可以在骨架网格组件上访问该对象。
	* 运行时设置的值与绘制的值相乘组合，得到最终的弹簧和阻尼强度。

### 2.5 自碰撞（Self Collision）

<div align=center>
<img src="/enclosures/2022-05-20/Cloth Map 1.5.png"/>
<p>Cloth Map 1.5</p>
</div>

展示启用或禁用 自碰撞 的示例，确保布料在移动过程中不会与其自身相交。

在*布料配置*的*自碰撞半径*中设置。

## 三、参考

* [布料内容示例 \| 虚幻引擎文档](https://docs.unrealengine.com/4.27/zh-CN/Resources/ContentExamples/Cloth/)

* [布料工具 \| 虚幻引擎文档](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Physics/Cloth/Overview/)



