---
layout: post
author: chaguake
title: UE Course - Blueprint To C++
categories: 
- Unreal Engine
tags: 
- UE Course
---

## 一、[将蓝图转换为C++](https://learn.unrealengine.com/course/3547921)

**将蓝图转换为C++**是虚幻引擎官方的一门课程，借助一个简单的蓝图游戏示例，演示了如何将蓝图类转换为C++类。

### 1.1 [创建C++基类](https://learn.unrealengine.com/course/3547921/module/6872680)

教程视频中使用UE4.23版本，在内容浏览器选项卡的*添加/导入*按钮中添加C++类，而4.26高版本则在*文件->项目->新建C++类*中添加C++类。

### 1.2 [UPROPERTY变量](https://learn.unrealengine.com/course/3547921/module/6872684)

```
UPROPERTY()
var varies;
```

UPROPERTY变量是在目标变量上方添加`UPROPERTY()`宏。

<div align=center>
<img src="/enclosures/2022-06-18/UPROPERTY变量访问权限属性.PNG"/>
<p>UPROPERTY变量访问权限属性</p>
</div>

### 1.3 [蓝图可调用的UFUNCTION](https://learn.unrealengine.com/course/3547921/module/6872685)

```
UFUNCTION(BlueprintCallable, BlueprintPure)
var function();
```

与UPROPERTY变量类似。

### 1.4 [蓝图可实现事件](https://learn.unrealengine.com/course/3547921/module/6872687)

蓝图可实现事件的关键字为`BlueprintImplementableEvent`，在C++中声明函数签名，然后在蓝图中实现，也可以被蓝图调用，只需添加关键字`BlueprintCallable`。

### 1.5 [蓝图本地事件](https://learn.unrealengine.com/course/3547921/module/6872688)

相对于蓝图可实现事件，蓝图本地事件允许在C++中实现函数体，但需要在函数名添加后缀`_Implementation`。然后在蓝图中重载，调用C++实现的函数。

## 二、参考

* [将蓝图转换为C++](https://learn.unrealengine.com/course/3547921)
