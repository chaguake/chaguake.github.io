---
layout: post
author: chaguake
title: Cycles Source Code - BVH
categories: 
- Cycles
tags: 
- Cycles Source Code
---

## 一、Bounding volume hierarchy（BVH）

在[Graphics Algorithm - Bounding Volume Hierarchy (BVH)](https://www.chaguake.com/graphics-algorithm-bounding-volume-hierarchy-bvh)一文中介绍了BVH算法。

## 二、Cycles BVH

### 2.1 Devices

Cycles 支持多种光线追踪加速结构，具体取决于设备。

* 用于CPU的[Embree](https://www.embree.org/)，对应源码的`BVHEmbree`类。

* OptiX BVH 用于 NVIDIA GPU 上的硬件光线追踪，对应源码的`BVHOptiX`类。

* 其他一切的自定义 BVH 实施，例如 CUDA 和 HIP 设备，对应源码的`BVH2`类。

自定义BVH在（快速移动）运动模糊方面的性能较差，因为仅支持原始级别的运动模糊，而不支持中间模式。

### 2.2 Intersection Types

需要使用`Ray Traces`的场景有：

* 最近命中的交叉点，用于相机和间接光线。记录一个交点，离射线起始位置最近的一个。

* 阴影交叉点。如果发现与已知具有不透明材料的图元相交，则立即停止遍历并认为光线完全被遮挡。如果找到与（可能）透明图元的交点，我们记录最多N个最近的透明交点，以评估它们的着色器。如果找到N个以上的透明交叉点，则将从更远的起点再次追踪光线。

* 次表面散射交叉点。与最近的命中相交类似，但仅在同一对象内相交。由于这个原因，具有次表面散射的对象总是被认为是“实例化的”，因此它们具有可以直接相交的专用BVH。

* 环境光遮挡着色器节点相交。与次表面散射交叉点相同，找到一个最近的命中。

* 斜角着色器节点相交。在当前着色点附近的同一对象内查找最多N个交点。如果发现超过N个交叉点，则使用蓄水池采样算法（reservoir sampling）来概率性地选择一个子集。

### 2.3 Self-Intersection Avoidance

从三角形或曲线开始的光线可能会自相交。为了避免这个问题，光线起始位置沿着几何法线得到一个小的偏移，类似于[A Fast and Robust Method for Avoiding Self-Intersection](https://link.springer.com/content/pdf/10.1007%2F978-1-4842-4427-2_6.pdf)。


### 2.4 自定义BVH实现

自定义BVH是基于表面积启发式（SAH）和空间分割构建的。构建性能通过分箱和多线程进行了优化。对于遍历，来自两个级别的节点仍然被打包到一个数组中。BVH遍历算法的实现方式是，在GPU上，即使两个线程处于不同的BVH级别，也可以连贯地执行BVH节点交叉。

## 三、cycles_bvh模块

<div align=center>
<img src="/enclosures/2022-05-15/call stack.PNG"/>
<p>Call Stack</p>
</div>

看上图的调用堆栈，先后经过`Session`类、`Scene`类和`GeometryManager`类，才进入到`cycles_bvh`模块。

### 3.1 params.h

定义了`cycles_bvh`模块所用到的一些数据结构。

* `BVHLayout`枚举了各种光线追踪加速结构。

* `BVH_TYPE_DYNAMIC`支持网格动态更新，`BVH_TYPE_STATIC`需要重新构建BVH树。

* `BVHParams`类保存了构建BVH树所用到的一些参数。

* `BVHReference`类提供封装对齐功能。

* `BVHRange`类封装了包围盒的范围值。

* `BVHSpatialBin`类保存了光线进出包围盒信息。

* `BVHSpatialStorage`类的想法是为空间拆分器提供特定于线程的存储。可以提前分配这个存储，避免在拆分过程中进行大量内存操作。

### 3.2 unaligned.h

`BVHUnaligned`类提供未对齐节点所需计算的功能。

### 3.3 sort.h

`bvh_reference_sort`函数用于对`BVHReference`类对象集合排序。当待排序元素数量大于`BVH_SORT_THRESHOLD`时，使用任务池来并发排序。


### 3.4 node.h

`InnerNode`类和`LeafNode`类都继承于`BVHNode`类。`InnerNode`类存储场景空间划分的包围盒信息，`LeafNode`类存储物体的包围盒信息。

### 3.5 split.h

对于场景空间划分，有三种方式：

`BVHObjectSplit`类基于物体的划分；

`BVHSpatialSplit`类基于空间的划分，物体可能会被分割在不同的包围盒中；

`BVHMixedSplit`类则是前两者的混合方式。

### 3.6 build.h

`BVHBuild`类为`BVH2`类的BVH构建器。`BVHBuild::run`为入口函数。

### 3.7 bvh.h

```
class BVH {
 public:
  BVHParams params;
  vector<Geometry *> geometry;
  vector<Object *> objects;

  static BVH *create(const BVHParams &params,
                     const vector<Geometry *> &geometry,
                     const vector<Object *> &objects,
                     Device *device);
  virtual ~BVH()
  {
  }

 protected:
  BVH(const BVHParams &params,
      const vector<Geometry *> &geometry,
      const vector<Object *> &objects);
};
```

`BVH`类提供了各种光线追踪加速结构的基类。

```
BVH *BVH::create(const BVHParams &params,
                 const vector<Geometry *> &geometry,
                 const vector<Object *> &objects,
                 Device *device)
{
  switch (params.bvh_layout) {
    case BVH_LAYOUT_BVH2:
      return new BVH2(params, geometry, objects);
    case BVH_LAYOUT_EMBREE:
#ifdef WITH_EMBREE
      return new BVHEmbree(params, geometry, objects);
#else
      break;
#endif
    case BVH_LAYOUT_OPTIX:
#ifdef WITH_OPTIX
      return new BVHOptiX(params, geometry, objects, device);
#else
      (void)device;
      break;
#endif
    case BVH_LAYOUT_METAL:
#ifdef WITH_METAL
      return bvh_metal_create(params, geometry, objects, device);
#else
      (void)device;
      break;
#endif
    case BVH_LAYOUT_MULTI_OPTIX:
    case BVH_LAYOUT_MULTI_METAL:
    case BVH_LAYOUT_MULTI_OPTIX_EMBREE:
    case BVH_LAYOUT_MULTI_METAL_EMBREE:
      return new BVHMulti(params, geometry, objects);
    case BVH_LAYOUT_NONE:
    case BVH_LAYOUT_ALL:
      break;
  }
  LOG(DFATAL) << "Requested unsupported BVH layout.";
  return NULL;
}
```

`BVH::create`函数根据`params.bvh_layout`创建对应的子类。

### 3.8 bvh2.h

> Typical BVH with each node having two children.

```
class BVH2 : public BVH {
public:
	void build(Progress &progress, Stats *stats);
	void refit(Progress &progress);
	PackedBVH pack;
	...
}
```

`BVH2`类`public`属性中，有两个成员函数和一个成员变量。

`build`函数创建BVH结构。调用`BVHBuild::run`函数构建BVH节点，返回根节点对象。然后将树结构数据打包成数组结构数据，存储在`pack`变量。

`refit`函数整修已有的BVH结构。

`pack`变量存储了打包好的BVH结构数据。


### 3.9 embree.h

// To do

### 3.10 optix.h

// To do

### 3.11 multi.h

// To do

## 四、参考

[1] [Cycles BVH - cycles wiki](https://wiki.blender.org/wiki/Source/Render/Cycles/BVH)

