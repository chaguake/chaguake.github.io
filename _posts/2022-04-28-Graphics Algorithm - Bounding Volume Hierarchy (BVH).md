---
layout: post
author: chaguake
title: Graphics Algorithm - Bounding Volume Hierarchy (BVH)
categories: 
- Graphics Algorithm
tags: 
- Bounding Volume Hierarchy
---

## 一、Bounding Volume Hierarchy (BVH)

<div align=center>
<img src="/enclosures/2022-04-28/Example_of_bounding_volume_hierarchy.png"/>
<p>An example of a bounding volume hierarchy using rectangles as bounding volumes</p>
</div>

BVH是一组几何对象的树形结构，树的Leaf Node为几何对象，树的Inner Node为包围盒。

BVH通常用于光线追踪，通过省略位于边界体积中且未被当前光线相交的几何对象来消除场景中潜在的交叉点候选对象。

## 二、kt_bvh

借助`kt_bvh`开源项目可以快速了解BVH算法怎么实现的，毕竟只有两个文件。

### 2.1 main函数

伪代码：

```
int main(int argc, char** _argv):
	load_model();
	bvh_build_intermediate();
	ray_tracing();
	free();
```

### 2.2 kt_bvh开源项目几个关键的类或结构体

#### 2.2.1 TracerCtx

```
struct TracerCtx
{
	~TracerCtx()
	{
		fast_obj_destroy(mesh);
		free16(bvh4);
		free16(pos_indices);
		free16(prim_id_buf);
		free16(image);
		free16(preprocessed_tris);
	}

	fastObjMesh* mesh = nullptr;
	PreprocessedTri* preprocessed_tris = nullptr;
	uint32_t* pos_indices = nullptr;
	uint32_t* prim_id_buf = nullptr;
	uint8_t* image = nullptr;

	union 
	{
		kt_bvh::BVH4Node* bvh4 = nullptr;
		kt_bvh::BVH2Node* bvh2;
	};

};
```


`TracerCtx`结构体保存了程序的上下文。

#### 2.2.2 kt_bvh::TriMesh

```
struct TriMesh
{
	enum class IndexType : uint32_t
	{
		UnIndexed,
		U16,
		U32
	};

	void set_vertices(float const* _vertices, uint32_t _vertex_stride, uint32_t _num_vertices)
	{
		vertices = _vertices;
		num_vertices = _num_vertices;
		vertex_stride = _vertex_stride;
	}

	void set_unindexed()
	{
		indices_u16 = nullptr;
		num_idx_prims = 0;
		index_type = IndexType::UnIndexed;
	}

	void set_indices(uint32_t const* _index_buffer, uint32_t _num_prims)
	{
		indices_u32 = _index_buffer;
		num_idx_prims = _num_prims;
		index_type = IndexType::U32;
	}

	void set_indices(uint16_t const* _index_buffer, uint32_t _num_prims)
	{
		indices_u16 = _index_buffer;
		num_idx_prims = _num_prims;
		index_type = IndexType::U16;
	}

	uint32_t total_prims() const
	{
		return index_type == IndexType::UnIndexed ? num_vertices / 3 : num_idx_prims;
	}

	// Vertex buffer, containing position (float3) data.
	float const* vertices;

	// Stride between float3 position elements in vertex buffer.
	uint32_t vertex_stride;

	// Total number of vertices.
	uint32_t num_vertices;

	// Index buffer.
	union
	{
		uint16_t const* indices_u16;
		uint32_t const* indices_u32;
	};

	// Data type of index buffer.
	IndexType index_type;

	// Number of primitives in index buffer, if one is used.
	uint32_t num_idx_prims;
};
```

`kt_bvh::TriMesh`结构体封装了模型的信息（例如：三角形和顶点位置）。

#### 2.2.3 kt_bvh::IntermediateBVH

```
struct IntermediateBVH
{
	MemArena arena;
	IntermediateBVHNode *root;
	uint32_t total_nodes;
	uint32_t total_leaf_nodes;

	dyn_pod_array<PrimitiveID> bvh_prim_id_list;
	uint32_t max_depth;

	BVHWidth width;
};
```

`kt_bvh::IntermediateBVH`结构体存储了BVH算法实现过程中的中间数据。

#### 2.2.4 kt_bvh::BVHBuildDesc

```
struct BVHBuildDesc
{
	static uint32_t const c_max_branching_factor = 4;

	static uint32_t const c_default_min_prims_per_leaf = 4;
	static uint32_t const c_default_max_prims_per_leaf = 8;
    static uint32_t const c_default_sah_buckets = 16;

	static uint32_t const c_max_sah_buckets = 32;

	uint32_t get_branching_factor() const
	{
		switch (width)
		{
			case BVHWidth::BVH2: return 2;
			case BVHWidth::BVH4: return 4;
		}

		KT_BVH_ASSERT(false); KT_BVH_UNREACHABLE;
	}

	// BVH build algorithm.
	BVHBuildType type = BVHBuildType::TopDownBinnedSAH;

	// Threshold of primitive to force leaf creation.
	uint32_t min_leaf_prims = c_default_min_prims_per_leaf;

	// Maximum amount of primitives per leaf, nodes will be further split to accommodate.
	uint32_t max_leaf_prims = c_default_max_prims_per_leaf;

	// Number of surface area heuristic binning buckets.
	uint32_t sah_buckets = c_default_sah_buckets;

	// Width of BVH.
	BVHWidth width = BVHWidth::BVH2;

	// Estimated cost of traversal for surface area heuristic (relative to intersection cost).
	float sah_traversal_cost = 0.85f;

    // Should SAH check all axis, or just major axis of AABB.
    bool sah_exhaustive_axis_test = true;
};
```

`kt_bvh::BVHBuildDesc`结构体存储了BVH构建的描述符信息。

#### 2.2.5 kt_bvh::BVHBuildResult

```
struct BVHBuildResult
{
	// Root of BVH tree.
	IntermediateBVHNode const* root;

	// Total amount of intermediate nodes in tree.
	uint32_t total_nodes;

	// Total amount of leaf nodes in tree.
	uint32_t total_leaf_nodes;

	// Total amount of interior nodes in tree.
	uint32_t total_interior_nodes;

	// Maximum depth of tree.
	uint32_t max_depth;

	// Primitive ID array that intermediate nodes point to.
	PrimitiveID const* prim_id_array;
	uint32_t prim_id_array_size;
};
```

`kt_bvh::BVHBuildResult`结构体存储了BVH构建的结果。

### 2.3 kt_bvh::bvh_build_intermediate

```
IntermediateBVH* bvh_build_intermediate(TriMesh const* _meshes, uint32_t _num_meshes, BVHBuildDesc const& _build_desc, AllocHooks* alloc_hooks)
{
	uint32_t total_prims = 0;
	for (uint32_t i = 0; i < _num_meshes; ++i)
	{
		total_prims += _meshes[i].total_prims();
	}

	if (!total_prims)
	{
		return nullptr;
	}
	...
}
```

获取所有网格的基元总数（也就是三角形数量）。

```
IntermediateBVH* bvh_build_intermediate(TriMesh const* _meshes, uint32_t _num_meshes, BVHBuildDesc const& _build_desc, AllocHooks* alloc_hooks)
{
	...
	AllocHooks hooks = alloc_hooks ? *alloc_hooks : malloc_hooks();

	IntermediateBVH* bvh_intermediate = nullptr;

	{
		MemArena arena;
		arena.init(hooks);
		bvh_intermediate = arena.alloc_zeroed<IntermediateBVH>();
		bvh_intermediate->arena = arena;
	}

	bvh_intermediate->width = _build_desc.width;

	BVHBuilderContext builder = {};

	builder.meshes = _meshes;
	builder.num_meshes = _num_meshes;
	builder.total_primitives = total_prims;
	builder.bvh = bvh_intermediate;
	builder.primitive_info = builder.arena().alloc_array_uninitialized<IntermediatePrimitive>(total_prims);
	builder.prim_ids = builder.arena().alloc_array_uninitialized<PrimitiveID>(total_prims);
	builder.build_desc = _build_desc;
	bvh_intermediate->bvh_prim_id_list.init(&bvh_intermediate->arena);

	bvh_intermediate->bvh_prim_id_list.ensure_cap(total_prims); // fixed capacity here, if we do spatial splits may need to increase.

	...
}
```

创建并初始化`BVHBuilderContext`对象和`IntermediateBVH`对象。

```
	...
	AABB enclosing_aabb, centroid_aabb;
	build_prim_info(builder, &enclosing_aabb, &centroid_aabb);
	...
```

生成enclosing_aabb包围盒和centroid_aabb包围盒。

```
	...
	builder.bvh->root = build_bvhn_recursive(builder, enclosing_aabb, centroid_aabb, 0, 0, total_prims);
	...
```

递归构建BVH树。

### 2.4 build_bvhn_recursive

**1 split_node**

```
	...
	uint32_t const middle_prim_split_idx = split_node(_ctx, _enclosing_aabb, _centroid_aabb, _prim_begin, _prim_end, &split_axis[0]);
	...
```

`split_node`函数根据两个parent AABB包围盒进行分割parent node。有两种分割方式：MedianSplit和TopDownBinnedSAH。

返回一个`_ctx.primitive_info`索引值，表示前`n`个基元都会在子节点的`_centroid_aabb`内。

```
static uint32_t split_node
(
	BVHBuilderContext& _ctx,
	AABB const& _enclosing_aabb,
	AABB const& _centroid_aabb,
	uint32_t _prim_begin,
	uint32_t _prim_end,
	uint32_t* o_split_axis
)
{
	uint32_t middle_prim_split_idx = c_invalid_split;

	switch (_ctx.build_desc.type)
	{
		case BVHBuildType::MedianSplit:
		{
			uint32_t const axis = aabb_major_axis(_centroid_aabb);
			float const midpoint = (_centroid_aabb.max * 0.5f + _centroid_aabb.min * 0.5f).data[axis];
			middle_prim_split_idx = partition_prim_id(_ctx, _prim_begin, _prim_end,
													   [midpoint, axis](IntermediatePrimitive const& _val) { return _val.origin.data[axis] < midpoint; });
			*o_split_axis = axis;
			if (middle_prim_split_idx == _prim_begin || middle_prim_split_idx == _prim_end)
			{
				// TODO: Sort?
				return (_prim_begin + _prim_end) / 2;
			}
			return middle_prim_split_idx;
		};

		case BVHBuildType::TopDownBinnedSAH:
		{
			SAHSplitResult const object_split = eval_sah_split(_ctx, _enclosing_aabb, _centroid_aabb, _prim_begin, _prim_end);

			if (!object_split.is_valid())
			{
				return c_invalid_split;
			}

			*o_split_axis = object_split.axis;

			middle_prim_split_idx = exec_sah_split(_ctx, object_split, _centroid_aabb, _prim_begin, _prim_end);
			KT_BVH_ASSERT(middle_prim_split_idx > _prim_begin && middle_prim_split_idx < _prim_end);
			return middle_prim_split_idx;
		} break;

	}

	KT_BVH_ASSERT(false); KT_BVH_UNREACHABLE;
}
```

**2 calc_enclosing_and_centroid_aabb**

```
	...
	PreSplitIntermediateBVHNode presplit_nodes[BVHBuildDesc::c_max_branching_factor];
	uint32_t num_children = 2;

	presplit_nodes[0].prim_begin = _prim_begin;
	presplit_nodes[0].prim_end = middle_prim_split_idx;
	calc_enclosing_and_centroid_aabb(_ctx, _prim_begin, middle_prim_split_idx, &presplit_nodes[0].enclosing_aabb, &presplit_nodes[0].centroid_aabb);

	presplit_nodes[1].prim_begin = middle_prim_split_idx;
	presplit_nodes[1].prim_end = _prim_end;
	calc_enclosing_and_centroid_aabb(_ctx, middle_prim_split_idx, _prim_end, &presplit_nodes[1].enclosing_aabb, &presplit_nodes[1].centroid_aabb);
	...
```

分别计算当前parent node的child node的AABB包围盒。

**3 while (num_children < branching_factor)**

```
	...
	uint32_t const branching_factor = _ctx.build_desc.get_branching_factor();
	KT_BVH_ASSERT(branching_factor <= BVHBuildDesc::c_max_branching_factor);

	while (num_children < branching_factor)
	{
		uint32_t best_leaf_idx = UINT32_MAX;
		float largest_surface_area = -FLT_MAX;

		for (uint32_t i = 0; i < num_children; ++i)
		{
			PreSplitIntermediateBVHNode const& split_node = presplit_nodes[i];
			float const split_surface_area = aabb_half_surface_area(split_node.enclosing_aabb);

			if (split_node.num_prims() > _ctx.build_desc.min_leaf_prims
				&& split_surface_area > largest_surface_area)
			{
				largest_surface_area = split_surface_area;
				best_leaf_idx = i;
			}
		}

		if (best_leaf_idx == UINT32_MAX)
		{
			// At this point all leafs fit so exit.
			break;
		}

		PreSplitIntermediateBVHNode& to_split = presplit_nodes[best_leaf_idx];
		uint32_t child_split_axis;
		uint32_t const child_middle_prim_split_idx = split_node(_ctx, to_split.enclosing_aabb, to_split.centroid_aabb, to_split.prim_begin, to_split.prim_end, &child_split_axis);

		if (child_middle_prim_split_idx == c_invalid_split)
		{
			break;
		}

		KT_BVH_ASSERT(num_children <= BVHBuildDesc::c_max_branching_factor);

		// Shuffle up to make room for this split and retain split_axis ordering (if we are splitting in the middle of the array).
		for (uint32_t split_insert = num_children - 1; split_insert > best_leaf_idx; --split_insert)
		{
			presplit_nodes[split_insert + 1] = presplit_nodes[split_insert];
			split_axis[split_insert] = split_axis[split_insert - 1];
		}


		uint32_t const split_begin = to_split.prim_begin;
		uint32_t const split_end = to_split.prim_end;

		KT_BVH_ASSERT(child_middle_prim_split_idx > split_begin && child_middle_prim_split_idx < split_end);

		presplit_nodes[best_leaf_idx].prim_begin = split_begin;
		presplit_nodes[best_leaf_idx].prim_end = child_middle_prim_split_idx;
		presplit_nodes[best_leaf_idx + 1].prim_begin = child_middle_prim_split_idx;
		presplit_nodes[best_leaf_idx + 1].prim_end = split_end;

		for (uint32_t leftright = 0; leftright < 2; ++leftright)
		{
			PreSplitIntermediateBVHNode& presplit = presplit_nodes[best_leaf_idx + leftright];
			calc_enclosing_and_centroid_aabb(_ctx, presplit.prim_begin, presplit.prim_end, &presplit.enclosing_aabb, &presplit.centroid_aabb);
		}

		split_axis[best_leaf_idx] = child_split_axis;

		++num_children;
	};
	...
```

兼容BVH4模式。

**4 build_bvhn_recursive**

```
	...
	IntermediateBVHNode* subtree_root = _ctx.new_node(_depth);
	static_assert(sizeof(subtree_root->split_axis) >= sizeof(split_axis), "Split axis arrays mismatch");
	memcpy(subtree_root->split_axis, split_axis, sizeof(split_axis));

	subtree_root->num_children = num_children;

	AABB subtree_aabb = aabb_invalid();

	for (uint32_t i = 0; i < num_children; ++i)
	{
		PreSplitIntermediateBVHNode const& split = presplit_nodes[i];
		IntermediateBVHNode* child = build_bvhn_recursive(_ctx, split.enclosing_aabb, split.centroid_aabb, _depth + 1, split.prim_begin, split.prim_end);
		subtree_root->children[i] = child;
		subtree_aabb = aabb_union(subtree_aabb, aabb_init(child->aabb_min, child->aabb_max));
	}

	node_copy_aabb(subtree_root, subtree_aabb);
	return subtree_root;
	...
```

下一次迭代构建BVH的工作。

### 2.5 build_prim_info_impl

```
static AABB aabb_union(AABB const& _lhs, AABB const& _rhs)
{
	return AABB{ min(_lhs.min, _rhs.min), max(_lhs.max, _rhs.max) };
}

static AABB aabb_expand(AABB const& _aabb, Vec3 const& _v)
{
	return AABB{ min(_aabb.min, _v), max(_aabb.max, _v) };
}

static Vec3 aabb_center(AABB const& _aabb)
{
	return _aabb.min * 0.5f + _aabb.max * 0.5f;
}

template <void (GetPrimT)(TriMesh const&, uint32_t, Vec3*, Vec3*, Vec3*)>
static void build_prim_info_impl(TriMesh const& _mesh, uint32_t _mesh_idx, IntermediatePrimitive* _prim_arr, PrimitiveID* _prim_id_arr, AABB* o_enclosing_aabb, AABB* o_centroid_aabb)
{
	AABB centroid_aabb = *o_centroid_aabb;
	AABB enclosing_aabb = *o_centroid_aabb;

	for (uint32_t i = 0; i < _mesh.total_prims(); ++i)
	{
		Vec3 tri[3];
		GetPrimT(_mesh, i, &tri[0], &tri[1], &tri[2]);
		_prim_arr->aabb = AABB{ tri[0], tri[0] };
		_prim_arr->aabb = aabb_expand(_prim_arr->aabb, tri[1]);
		_prim_arr->aabb = aabb_expand(_prim_arr->aabb, tri[2]);
		_prim_arr->origin = aabb_center(_prim_arr->aabb);
		_prim_id_arr->mesh_idx = _mesh_idx;
		_prim_id_arr->mesh_prim_idx = i;

		enclosing_aabb = aabb_union(enclosing_aabb, _prim_arr->aabb);
		centroid_aabb = aabb_expand(centroid_aabb, _prim_arr->origin);

		++_prim_arr;
		++_prim_id_arr;
	}

	*o_centroid_aabb = centroid_aabb;
	*o_enclosing_aabb = enclosing_aabb;
}
```

**enclosing_aabb**

`enclosing_aabb`计算出所有基元各个轴向的最小值和最大值作为包围盒的`min`和`max`。

**centroid_aabb**

`centroid_aabb`计算出每个基元各个轴向的最小值和最大值之后，取其平均值，然后判断所有计算平均值的最大值和最小值作为包围盒的`min`和`max`。

这种包围盒去掉了更多的空白空间，代价是网格有些三角形的一半不在包围盒内。

### 2.6 BVHBuildType::MedianSplit

`aabb_major_axis`函数判断parent node的三个轴向大小，最大值的那个轴向作为划分child node的方向。

```

// Partition array such that all passing the predicate come first.
template <typename PredT>
static uint32_t partition_prim_id(BVHBuilderContext& _ctx, uint32_t _begin, uint32_t _end, PredT _pred)
{
	IntermediatePrimitive* prims = _ctx.primitive_info;
	PrimitiveID* ids = _ctx.prim_ids;

	uint32_t swap_idx = _begin;

	while (swap_idx != _end && _pred(prims[swap_idx]))
	{
		++swap_idx;
	}

	if (swap_idx != _end)
	{
		uint32_t cur_idx = swap_idx + 1;

		while (cur_idx != _end)
		{
			if (_pred(prims[cur_idx]))
			{
				swap(prims[cur_idx], prims[swap_idx]);
				swap(ids[cur_idx], ids[swap_idx]);
				++swap_idx;
			}
			++cur_idx;
		}
	}

	return swap_idx;
}
```

`partition_prim_id`遍历每一个基元，并且交换基元数组的元素位置，使得基元数组前`n`个都会比切割线的值小。

这种划分是不均衡的，很容易导致二叉树的效率发挥不出来（退化成类链表）。

### 2.7 BVHBuildType::TopDownBinnedSAH

<div align=center>
<img src="/enclosures/2022-04-28/slide_024.png"/>
<p>SAH</p>
</div>

```
static SAHSplitResult eval_sah_split
(
	BVHBuilderContext& _ctx,
	AABB const& _enclosing_aabb,
	AABB const& _centroid_aabb,
	uint32_t _prim_begin,
	uint32_t _prim_end
)
{
	SAHSplitResult best_split_result;
	SAHBucketingInfo axis_buckets[3];

	uint32_t const num_buckets = _ctx.build_desc.sah_buckets;
	uint32_t const num_splits = num_buckets - 1;

	Vec3 const split_axis_len = (_centroid_aabb.max - _centroid_aabb.min);

	Vec3 const project_dim_constant = vec3_splat(float(num_buckets)) / split_axis_len;
	Vec3 const bucket_max = vec3_splat(float(num_buckets - 1));

	for (uint32_t prim_idx = _prim_begin; prim_idx < _prim_end; ++prim_idx)
	{
		IntermediatePrimitive const& prim = _ctx.primitive_info[prim_idx];
		Vec3 const bucket3 = min(bucket_max, (prim.origin - _centroid_aabb.min) * project_dim_constant);

		uint32_t const bucket0 = uint32_t(bucket3.x);
		uint32_t const bucket1 = uint32_t(bucket3.y);
		uint32_t const bucket2 = uint32_t(bucket3.z);

		axis_buckets[0].bucket_num_prims[bucket0]++;
		axis_buckets[1].bucket_num_prims[bucket1]++;
		axis_buckets[2].bucket_num_prims[bucket2]++;

		axis_buckets[0].bucket_bounds[bucket0] = aabb_union(prim.aabb, axis_buckets[0].bucket_bounds[bucket0]);
		axis_buckets[1].bucket_bounds[bucket1] = aabb_union(prim.aabb, axis_buckets[1].bucket_bounds[bucket1]);
		axis_buckets[2].bucket_bounds[bucket2] = aabb_union(prim.aabb, axis_buckets[2].bucket_bounds[bucket2]);
	}


	for (uint32_t split_axis = 0; split_axis < 3; ++split_axis)
	{
		if (split_axis_len.data[split_axis] <= 0.0001f)
		{
			continue;
		}

		SAHBucketingInfo& bucket_info = axis_buckets[split_axis];

		bucket_info.forward_split_bounds[0] = bucket_info.bucket_bounds[0];
		bucket_info.forward_prim_count[0] = bucket_info.bucket_num_prims[0];

		for (int32_t i = 1; i < int32_t(num_splits); ++i)
		{
			bucket_info.forward_split_bounds[i] = aabb_union(bucket_info.forward_split_bounds[i - 1], bucket_info.bucket_bounds[i]);
			bucket_info.forward_prim_count[i] = bucket_info.forward_prim_count[i - 1] + bucket_info.bucket_num_prims[i];
		}

		AABB incremental_reverse_aabb = aabb_invalid();
		uint32_t incremental_reverse_prim_count = 0;
		float const root_surface_area = aabb_half_surface_area(_enclosing_aabb);

		uint32_t best_split = UINT32_MAX;
		float best_cost = FLT_MAX;

		float const traversal_cost = _ctx.build_desc.sah_traversal_cost;

		uint32_t const nprims = _prim_end - _prim_begin;

		for (int32_t i = int32_t(num_splits) - 1; i >= 0; --i)
		{
			incremental_reverse_aabb = aabb_union(incremental_reverse_aabb, bucket_info.bucket_bounds[i + 1]);
			incremental_reverse_prim_count += bucket_info.bucket_num_prims[i + 1];

			uint32_t const countA = bucket_info.forward_prim_count[i];
			uint32_t const countB = incremental_reverse_prim_count;

			KT_BVH_ASSERT(countA + countB == nprims);

			if (countA == 0 || countB == 0)
			{
				continue;
			}

			AABB const& forward_bounds = bucket_info.forward_split_bounds[i];

			float const sa = aabb_half_surface_area(forward_bounds);
			float const sb = aabb_half_surface_area(incremental_reverse_aabb);
			float const cost = traversal_cost + (sa * countA + sb * countB) / root_surface_area;

			if (cost < best_cost)
			{
				best_cost = cost;
				best_split = i;
			}
		}

		if (best_split == UINT32_MAX)
		{
			continue;
		}

		SAHSplitResult result;

		result.axis = split_axis;
		result.split_idx = best_split;
		result.sah_cost = best_cost;
		result.sah_leaf_cost = float(nprims);

		if (result.is_valid() && result.best_sah_cost() < best_split_result.best_sah_cost())
		{
			best_split_result = result;
		}
	}

	return best_split_result;
}
```

设置桶的数量为16，遍历每个基元，分别计算各个轴向属于哪个桶。

遍历每个轴向，计算SAT。数据存储在`forward_split_bounds`变量和`forward_prim_count`变量。

从后往前遍历桶数组，计算当前划分的代价，对比记录最小代价。

```
uint32_t exec_sah_split(BVHBuilderContext& _ctx, SAHSplitResult const& _result, AABB const& _centroid_aabb, uint32_t _prim_begin, uint32_t _prim_end)
{
	uint32_t const nbuckets = _ctx.build_desc.sah_buckets;
	float const split_axis_len = (_centroid_aabb.max - _centroid_aabb.min).data[_result.axis];
	float const project_dim_constant = nbuckets / split_axis_len;

	float const bucket_max = float(nbuckets - 1);

	uint32_t const mid_idx = partition_prim_id(_ctx, _prim_begin, _prim_end,
										   [&_result, nbuckets, &_centroid_aabb, project_dim_constant, bucket_max](IntermediatePrimitive const& _prim) -> bool
	{
		float const project_to_dim = (_prim.origin - _centroid_aabb.min).data[_result.axis] * project_dim_constant;
		uint32_t const bucket = uint32_t(min(bucket_max, project_to_dim));
		return bucket <= _result.split_idx;
	});

	return mid_idx;
}
```

操作与`BVHBuildType::MedianSplit`的一致。

### 2.8 ctx.prim_id_buf与ctx.preprocessed_tris

```
	...
	// Write out primitive id without mesh id.
		ctx.prim_id_buf = (uint32_t*)malloc16(sizeof(uint32_t) * result.prim_id_array_size);
		for (uint32_t i = 0; i < result.prim_id_array_size; ++i)
		{
			ctx.prim_id_buf[i] = result.prim_id_array[i].mesh_prim_idx;
		}

		// preprocess tris for moller trumbore intersection

		ctx.preprocessed_tris = (PreprocessedTri*)malloc16(sizeof(PreprocessedTri) * result.prim_id_array_size * 3);

		uint32_t* prim_begin = ctx.prim_id_buf;
		uint32_t* prim_end = prim_begin + result.prim_id_array_size;
		PreprocessedTri* tri_write = ctx.preprocessed_tris;
		while (prim_begin != prim_end)
		{
			uint32_t const face_idx = *prim_begin++;
			Vec3 const& v0 = *(((Vec3*)ctx.mesh->positions) + ctx.mesh->indices[face_idx * 3].p);
			Vec3 const& v1 = *(((Vec3*)ctx.mesh->positions) + ctx.mesh->indices[face_idx * 3 + 1].p);
			Vec3 const& v2 = *(((Vec3*)ctx.mesh->positions) + ctx.mesh->indices[face_idx * 3 + 2].p);

			tri_write->v0 = v0;
			tri_write->v01 = v1 - v0;
			tri_write->v02 = v2 - v0;
			++tri_write;
		}
	...
```

保存每个基元的顶点位置。


## 三、参考

* [Bounding volume hierarchy - wiki](https://en.m.wikipedia.org/wiki/Bounding_volume_hierarchy)

* [kt_bvh - github](https://github.com/karltechno/kt_bvh)

* [Accelerating Geometric Queries](http://15462.courses.cs.cmu.edu/fall2015/lecture/acceleration/slide_024)

* [PBRT-E4.3-层次包围体(BVH)（一）](https://zhuanlan.zhihu.com/p/50720158)