---
layout: post
author: chaguake
title: Cycles Source Code - BVH2
categories: 
- Cycles
tags: 
- Cycles Source Code
---

> Typical BVH with each node having two children.

<div align=center>
<img src="/enclosures/2022-04-25/Cycles-modules-architecture.png"/>
<p>Cycles modules architecture</p>
</div>


## 一、BVH相关函数的调用过程

以`cycles.exe --samples 10 --output ./cycles_image.png ../../../examples/scene_monkey.xml`为例，直接快进到`BVH2::build`函数。

```
void BVH2::build(Progress &progress, Stats *)
{
	/* build nodes */
	BVHBuild bvh_build(objects,
						 pack.prim_type,
						 pack.prim_index,
						 pack.prim_object,
						 pack.prim_time,
						 params,
						 progress);
	BVHNode *bvh2_root = bvh_build.run();

	...

	/* BVH builder returns tree in a binary mode (with two children per inner
	* node. Need to adopt that for a wider BVH implementations. */
	BVHNode *root = widen_children_nodes(bvh2_root);
	if (root != bvh2_root) {
	  bvh2_root->deleteSubtree();
	}

	...

	/* pack triangles */
	progress.set_substatus("Packing BVH triangles and strands");
	pack_primitives();

	...

	/* pack nodes */
	progress.set_substatus("Packing BVH nodes");
	pack_nodes(root);

	/* free build nodes */
	root->deleteSubtree();
}
```

### 1.1 build nodes

使用`BVHBuild`封装类对象的`run`函数来进行创建`BVHNode`对象。

```
BVHNode *BVHBuild::run()
{
  BVHRange root;

  /* add references */
  add_references(root);

  if (progress.get_cancel())
    return NULL;

  /* init spatial splits */
  if (params.top_level) {
    /* NOTE: Technically it is supported by the builder but it's not really
     * optimized for speed yet and not really clear yet if it has measurable
     * improvement on render time. Needs some extra investigation before
     * enabling spatial split for top level BVH.
     */
    params.use_spatial_split = false;
  }

  spatial_min_overlap = root.bounds().safe_area() * params.spatial_split_alpha;
  spatial_free_index = 0;

  need_prim_time = params.use_motion_steps();

  /* init progress updates */
  double build_start_time;
  build_start_time = progress_start_time = time_dt();
  progress_count = 0;
  progress_total = references.size();
  progress_original_total = progress_total;

  prim_type.resize(references.size());
  prim_index.resize(references.size());
  prim_object.resize(references.size());
  if (need_prim_time) {
    prim_time.resize(references.size());
  }
  else {
    prim_time.resize(0);
  }

  /* build recursively */
  BVHNode *rootnode;

  if (params.use_spatial_split) {
    /* Perform multithreaded spatial split build. */
    BVHSpatialStorage *local_storage = &spatial_storage.local();
    rootnode = build_node(root, references, 0, local_storage);
    task_pool.wait_work();
  }
  else {
    /* Perform multithreaded binning build. */
    BVHObjectBinning rootbin(root, (references.size()) ? &references[0] : NULL);
    rootnode = build_node(rootbin, 0);
    task_pool.wait_work();
  }

  /* clean up temporary memory usage by threads */
  spatial_storage.clear();

  /* delete if we canceled */
  if (rootnode) {
    if (progress.get_cancel()) {
      rootnode->deleteSubtree();
      rootnode = NULL;
      VLOG(1) << "BVH build cancelled.";
    }
    else {
      /*rotate(rootnode, 4, 5);*/
      rootnode->update_visibility();
      rootnode->update_time();
    }
  }

  return rootnode;
}
```

`BVHBuild::run`函数有两个关键的临时变量：`BVHRange root;`和`BVHNode *rootnode;`。

`BVHRange`类的作用：构建过程中使用的构建范围，用于指示基本体子集的边界和在参考数组中的位置，再次使用技巧将整数打包到BoundBox中，以便对齐。

#### 1.1.1 add references

```
void BVHBuild::add_references(BVHRange &root)
{
  /* reserve space for references */
  size_t num_alloc_references = 0;

  foreach (Object *ob, objects) {
    if (params.top_level) {
      if (!ob->is_traceable()) {
        continue;
      }
      if (!ob->get_geometry()->is_instanced()) {
        num_alloc_references += count_primitives(ob->get_geometry());
      }
      else {
        num_alloc_references++;
      }
    }
    else {
      num_alloc_references += count_primitives(ob->get_geometry());
    }
  }

  references.reserve(num_alloc_references);

  /* add references from objects */
  BoundBox bounds = BoundBox::empty, center = BoundBox::empty;
  int i = 0;

  foreach (Object *ob, objects) {
    if (params.top_level) {
      if (!ob->is_traceable()) {
        ++i;
        continue;
      }
      if (!ob->get_geometry()->is_instanced())
        add_reference_geometry(bounds, center, ob->get_geometry(), i);
      else
        add_reference_object(bounds, center, ob, i);
    }
    else
      add_reference_geometry(bounds, center, ob->get_geometry(), i);

    i++;

    if (progress.get_cancel())
      return;
  }

  /* happens mostly on empty meshes */
  if (!bounds.valid())
    bounds.grow(zero_float3());

  root = BVHRange(bounds, center, 0, references.size());
}
```

`BVHBuild::add_references`函数大致做两件事，然后返回一个新的`BVHRange`对象：

##### 1.1.1.1 reserve space for references

遍历每一个物体，然后判断其是否为实例（is_instanced），最后根据物体类型确认其基元数量。对于网格与体积，则是获取其网格三角形数量；对于毛发，则是获取其曲线段（curve segments）；对于点云，则是获取其点数量。如果物体是实例，则直接+1。

##### 1.1.1.2 add references from objects

遍历每一个物体，然后判断其是否为实例（is_instanced），来决定调用`add_reference_geometry`函数还是`add_reference_object`函数。

###### 1.1.1.2.1 BVHBuild::add_reference_geometry

```
void BVHBuild::add_reference_geometry(BoundBox &root,
                                      BoundBox &center,
                                      Geometry *geom,
                                      int object_index)
{
  if (geom->geometry_type == Geometry::MESH || geom->geometry_type == Geometry::VOLUME) {
    Mesh *mesh = static_cast<Mesh *>(geom);
    add_reference_triangles(root, center, mesh, object_index);
  }
  else if (geom->geometry_type == Geometry::HAIR) {
    Hair *hair = static_cast<Hair *>(geom);
    add_reference_curves(root, center, hair, object_index);
  }
  else if (geom->geometry_type == Geometry::POINTCLOUD) {
    PointCloud *pointcloud = static_cast<PointCloud *>(geom);
    add_reference_points(root, center, pointcloud, object_index);
  }
}

```

不同的物体类型调用不同的`add_reference_`函数。

####### 1.1.1.2.1.1 BVHBuild::add_reference_triangles

这里先不考虑运动模糊的网格。

```
void BVHBuild::add_reference_triangles(BoundBox &root,
                                       BoundBox &center,
                                       Mesh *mesh,
                                       int object_index)
{
  const PrimitiveType primitive_type = mesh->primitive_type();
  const size_t num_triangles = mesh->num_triangles();
  for (uint j = 0; j < num_triangles; j++) {
    Mesh::Triangle t = mesh->get_triangle(j);
    const float3 *verts = &mesh->verts[0];
    if (attr_mP == NULL) {
      BoundBox bounds = BoundBox::empty;
      t.bounds_grow(verts, bounds);
      if (bounds.valid() && t.valid(verts)) {
        references.push_back(BVHReference(bounds, j, object_index, primitive_type));
        root.grow(bounds);
        center.grow(bounds.center2());
      }
    }else 
	{ ... }
  }
```

遍历网格每个三角形，创建三角形的包围盒（BoundBox），封装一个`BVHReference`对象，添加到references容器中。

毛发与点云的操作类似。

###### 1.1.1.2.2 BVHBuild::add_reference_object

```
void BVHBuild::add_reference_object(BoundBox &root, BoundBox &center, Object *ob, int i)
{
  references.push_back(BVHReference(ob->bounds, -1, i, 0));
  root.grow(ob->bounds);
  center.grow(ob->bounds.center2());
}
```

对于示例物体来说，是一个整体。

#### 1.1.2 init spatial splits

`params`的`top_level`字段决定了BVH是基于空间划分还是基于网格划分。

#### 1.1.3 init progress updates

初始化进度更新的输出。

#### 1.1.4 build recursively

```
  if (params.use_spatial_split) {
    /* Perform multithreaded spatial split build. */
    BVHSpatialStorage *local_storage = &spatial_storage.local();
    rootnode = build_node(root, references, 0, local_storage);
    task_pool.wait_work();
  }
  else {
    /* Perform multithreaded binning build. */
    BVHObjectBinning rootbin(root, (references.size()) ? &references[0] : NULL);
    rootnode = build_node(rootbin, 0);
    task_pool.wait_work();
  }
```

`BVHObjectBinning`对象使用了Surface Area Heuristic（SAH）的划分算法。

### 1.2 Adopt the BVH builder has returned tree

> BVH builder returns tree in a binary mode (with two children per inner node. Need to adopt that for a wider BVH implementations.

```
void BVH2::build(Progress &progress, Stats *)
{
	...
	BVHNode *root = widen_children_nodes(bvh2_root);
    if (root != bvh2_root) {
      bvh2_root->deleteSubtree();
    }

    if (progress.get_cancel()) {
    if (root != NULL) {
      root->deleteSubtree();
    }
    return;
  }
	...
}
```

意义不明。

### 1.3 pack triangles

```
void BVH2::pack_primitives()
{
  const size_t tidx_size = pack.prim_index.size();
  /* Reserve size for arrays. */
  pack.prim_visibility.clear();
  pack.prim_visibility.resize(tidx_size);
  /* Fill in all the arrays. */
  for (unsigned int i = 0; i < tidx_size; i++) {
    if (pack.prim_index[i] != -1) {
      int tob = pack.prim_object[i];
      Object *ob = objects[tob];
      pack.prim_visibility[i] = ob->visibility_for_tracing();
    }
    else {
      pack.prim_visibility[i] = 0;
    }
  }
}
```

填充`pack`对象的`prim_visibility`字段，存储物体是否可以光线追踪的值。

### 1.4 pack nodes

```
void BVH2::pack_leaf(const BVHStackEntry &e, const LeafNode *leaf)
{
  assert(e.idx + BVH_NODE_LEAF_SIZE <= pack.leaf_nodes.size());
  float4 data[BVH_NODE_LEAF_SIZE];
  memset(data, 0, sizeof(data));
  if (leaf->num_triangles() == 1 && pack.prim_index[leaf->lo] == -1) {
    /* object */
    data[0].x = __int_as_float(~(leaf->lo));
    data[0].y = __int_as_float(0);
  }
  else {
    /* triangle */
    data[0].x = __int_as_float(leaf->lo);
    data[0].y = __int_as_float(leaf->hi);
  }
  data[0].z = __uint_as_float(leaf->visibility);
  if (leaf->num_triangles() != 0) {
    data[0].w = __uint_as_float(pack.prim_type[leaf->lo]);
  }

  memcpy(&pack.leaf_nodes[e.idx], data, sizeof(float4) * BVH_NODE_LEAF_SIZE);
}
```

对于叶子节点调用`BVH2::pack_leaf`。

```
void BVH2::pack_unaligned_node(int idx,
                               const Transform &aligned_space0,
                               const Transform &aligned_space1,
                               const BoundBox &bounds0,
                               const BoundBox &bounds1,
                               int c0,
                               int c1,
                               uint visibility0,
                               uint visibility1)
{
  assert(idx + BVH_UNALIGNED_NODE_SIZE <= pack.nodes.size());
  assert(c0 < 0 || c0 < pack.nodes.size());
  assert(c1 < 0 || c1 < pack.nodes.size());

  float4 data[BVH_UNALIGNED_NODE_SIZE];
  Transform space0 = BVHUnaligned::compute_node_transform(bounds0, aligned_space0);
  Transform space1 = BVHUnaligned::compute_node_transform(bounds1, aligned_space1);
  data[0] = make_float4(__int_as_float(visibility0 | PATH_RAY_NODE_UNALIGNED),
                        __int_as_float(visibility1 | PATH_RAY_NODE_UNALIGNED),
                        __int_as_float(c0),
                        __int_as_float(c1));

  data[1] = space0.x;
  data[2] = space0.y;
  data[3] = space0.z;
  data[4] = space1.x;
  data[5] = space1.y;
  data[6] = space1.z;

  memcpy(&pack.nodes[idx], data, sizeof(float4) * BVH_UNALIGNED_NODE_SIZE);
}
void BVH2::pack_aligned_node(int idx,
                             const BoundBox &b0,
                             const BoundBox &b1,
                             int c0,
                             int c1,
                             uint visibility0,
                             uint visibility1)
{
  assert(idx + BVH_NODE_SIZE <= pack.nodes.size());
  assert(c0 < 0 || c0 < pack.nodes.size());
  assert(c1 < 0 || c1 < pack.nodes.size());

  int4 data[BVH_NODE_SIZE] = {
      make_int4(
          visibility0 & ~PATH_RAY_NODE_UNALIGNED, visibility1 & ~PATH_RAY_NODE_UNALIGNED, c0, c1),
      make_int4(__float_as_int(b0.min.x),
                __float_as_int(b1.min.x),
                __float_as_int(b0.max.x),
                __float_as_int(b1.max.x)),
      make_int4(__float_as_int(b0.min.y),
                __float_as_int(b1.min.y),
                __float_as_int(b0.max.y),
                __float_as_int(b1.max.y)),
      make_int4(__float_as_int(b0.min.z),
                __float_as_int(b1.min.z),
                __float_as_int(b0.max.z),
                __float_as_int(b1.max.z)),
  };

  memcpy(&pack.nodes[idx], data, sizeof(int4) * BVH_NODE_SIZE);
}
```

对于非叶子节点调用`BVH2::pack_aligned_node`或`BVH2::pack_unaligned_node`。

到这里就完成了将BVH树结构转换成多个数组结构（`PackedBVH`对象）。

```
/* Packed BVH
 *
 * BVH stored as it will be used for traversal on the rendering device. */

struct PackedBVH {
  /* BVH nodes storage, one node is 4x int4, and contains two bounding boxes,
   * and child, triangle or object indexes depending on the node type */
  array<int4> nodes;
  /* BVH leaf nodes storage. */
  array<int4> leaf_nodes;
  /* object index to BVH node index mapping for instances */
  array<int> object_node;
  /* primitive type - triangle or strand */
  array<int> prim_type;
  /* Visibility visibilities for primitives. */
  array<uint> prim_visibility;
  /* mapping from BVH primitive index to true primitive index, as primitives
   * may be duplicated due to spatial splits. -1 for instances. */
  array<int> prim_index;
  /* mapping from BVH primitive index, to the object id of that primitive. */
  array<int> prim_object;
  /* Time range of BVH primitive. */
  array<float2> prim_time;

  /* index of the root node. */
  int root_index;

  PackedBVH()
  {
    root_index = 0;
  }
};
```

## 二、参考

[1] [pbr - 4.3 Bounding Volume Hierarchies](https://www.pbr-book.org/3ed-2018/Primitives_and_Intersection_Acceleration/Bounding_Volume_Hierarchies).

[2] [A Survey on Bounding Volume Hierarchies for Ray Tracing](https://meistdan.github.io/publications/bvh_star/paper.pdf).

[3] [SAT0: Surface-Area Traversal Order for Ray Tracing](https://nahjaeho.github.io/papers/CGF14/SATO2.pdf).

[4] [Spatial Splits in Bounding Volume Hierarchies - Nvidia](https://www.nvidia.com/docs/IO/77714/sbvh.pdf).

