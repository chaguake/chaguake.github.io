---
layout: post
title: Vulkan Basic Learning
categories: 
- Vulkan
tags: 
- Vulkan Basic
---

���İ���Github��[Vulkan��ԴC++ʾ��](https://github.com/SaschaWillems/Vulkan)��˳��ѧϰ�����岿����뿴��Github��Ŀ��README.md��

## Examples

[����](https://github.com/SaschaWillems/Vulkan#examples)

### һ��Basics

#### 1 First triangle

> Basic and verbose example for getting a colored triangle rendered to the screen using Vulkan. This is meant as a starting point for learning Vulkan from the ground up. A huge part of the code is boilerplate that is abstracted away in later examples.

һ���򵥵����������ʾ������Ҫ�ĵط�������һ��ʾ�������ܡ�


##### 1.1 Main��ں���

```
VulkanExample *vulkanExample;
LRESULT CALLBACK WndProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	if (vulkanExample != NULL)
	{
		vulkanExample->handleMessages(hWnd, uMsg, wParam, lParam);
	}
	return (DefWindowProc(hWnd, uMsg, wParam, lParam));
}
int APIENTRY WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR pCmdLine, int nCmdShow)
{
	for (size_t i = 0; i < __argc; i++) { VulkanExample::args.push_back(__argv[i]); };
	vulkanExample = new VulkanExample();
	vulkanExample->initVulkan();
	vulkanExample->setupWindow(hInstance, WndProc);
	vulkanExample->prepare();
	vulkanExample->renderLoop();
	delete(vulkanExample);
	return 0;
}
```

`WinMain`��������һ��VulkanExample�������ε������伸����Ա�����������Ⱦ������

##### 1.2 VulkanExampleBase::VulkanExampleBase���캯��

`enableValidation`����Ϊ`true`ʱ��������Console���ڣ����忴`vulkandebug.cpp`��

�ú�����Ҫ��һЩ�����в����Ľ���������

##### 1.3 VulkanExampleBase::initVulkan����

����`vkCreateInstance`��������`VkInstance`ʵ������

����`VkPhysicalDevice`�����豸�����`vks::VulkanDevice`�߼��豸����

�����豸���С����ӽ������Լ������ź�����

##### 1.4 VulkanExample��������ؼ�����

* `prepare`����������`VulkanExampleBase::prepare`���������ҳ�ʼ����Ŀ������Դ��׼��Buffers����Ⱦ���ߡ��������Լ�Command Buffers�ȵȡ�

* `render`��������״̬�����ύ��Ⱦ������ֻ��档

##### 1.5 Semaphores��Fences

```
// Create the Vulkan synchronization primitives used in this example
void prepareSynchronizationPrimitives()
{
	// Semaphores (Used for correct command ordering)
	VkSemaphoreCreateInfo semaphoreCreateInfo = {};
	semaphoreCreateInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;
	semaphoreCreateInfo.pNext = nullptr;

	// Semaphore used to ensures that image presentation is complete before starting to submit again
	VK_CHECK_RESULT(vkCreateSemaphore(device, &semaphoreCreateInfo, nullptr, &presentCompleteSemaphore));

	// Semaphore used to ensures that all commands submitted have been finished before submitting the image to the queue
	VK_CHECK_RESULT(vkCreateSemaphore(device, &semaphoreCreateInfo, nullptr, &renderCompleteSemaphore));

	// Fences (Used to check draw command buffer completion)
	VkFenceCreateInfo fenceCreateInfo = {};
	fenceCreateInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
	// Create in signaled state so we don't wait on first render of each command buffer
	fenceCreateInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;
	waitFences.resize(drawCmdBuffers.size());
	for (auto& fence : waitFences)
	{
		VK_CHECK_RESULT(vkCreateFence(device, &fenceCreateInfo, nullptr, &fence));
	}
}

void draw()
{
	// Get next image in the swap chain (back/front buffer)
	VK_CHECK_RESULT(swapChain.acquireNextImage(presentCompleteSemaphore, &currentBuffer));

	// Use a fence to wait until the command buffer has finished execution before using it again
	VK_CHECK_RESULT(vkWaitForFences(device, 1, &waitFences[currentBuffer], VK_TRUE, UINT64_MAX));
	VK_CHECK_RESULT(vkResetFences(device, 1, &waitFences[currentBuffer]));

	// Pipeline stage at which the queue submission will wait (via pWaitSemaphores)
	VkPipelineStageFlags waitStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
	// The submit info structure specifies a command buffer queue submission batch
	VkSubmitInfo submitInfo = {};
	submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
	submitInfo.pWaitDstStageMask = &waitStageMask;               // Pointer to the list of pipeline stages that the semaphore waits will occur at
	submitInfo.pWaitSemaphores = &presentCompleteSemaphore;      // Semaphore(s) to wait upon before the submitted command buffer starts executing
	submitInfo.waitSemaphoreCount = 1;                           // One wait semaphore
	submitInfo.pSignalSemaphores = &renderCompleteSemaphore;     // Semaphore(s) to be signaled when command buffers have completed
	submitInfo.signalSemaphoreCount = 1;                         // One signal semaphore
	submitInfo.pCommandBuffers = &drawCmdBuffers[currentBuffer]; // Command buffers(s) to execute in this batch (submission)
	submitInfo.commandBufferCount = 1;                           // One command buffer

	// Submit to the graphics queue passing a wait fence
	VK_CHECK_RESULT(vkQueueSubmit(queue, 1, &submitInfo, waitFences[currentBuffer]));

	// Present the current buffer to the swap chain
	// Pass the semaphore signaled by the command buffer submission from the submit info as the wait semaphore for swap chain presentation
	// This ensures that the image is not presented to the windowing system until all commands have been submitted
	VkResult present = swapChain.queuePresent(queue, currentBuffer, renderCompleteSemaphore);
	if (!((present == VK_SUCCESS) || (present == VK_SUBOPTIMAL_KHR))) {
		VK_CHECK_RESULT(present);
	}

}
```

��Vulkan API�ĵ�������Synchronization and Cache Control��

###### 1.5.1 Vulkan�ṩ��5����ʽͬ������

* Fences

> Fences can be used to communicate to the host that execution of some task on the device has completed.

Χ����Fences������������ͨ�ţ�˵��GPU�豸�������Ѿ�ִ����ɡ�

ʾ����Ϊÿһ��Frame Buffer����һ��fence����`draw`�����е��ÿ�����Ⱦ˳��

```
VK_CHECK_RESULT(vkWaitForFences(device, 1, &waitFences[currentBuffer], VK_TRUE, UINT64_MAX));
VK_CHECK_RESULT(vkResetFences(device, 1, &waitFences[currentBuffer]));
...
VK_CHECK_RESULT(vkQueueSubmit(queue, 1, &submitInfo, waitFences[currentBuffer]));
```

�����ڴ���`VkFence`����ʱ��`VkFenceCreateInfo`�ṹ���`flags`�ֶ�����Ϊ`VK_FENCE_CREATE_SIGNALED_BIT`�����Ե�һ�ε�Draw���ᱻ������

`vkQueueSubmit`���ĸ�����ʹ����Fences����ô��ȷ����ǰ�ύ�Ķ���ִ������������󷢳��źš�

* Semaphores

> Semaphores can be used to control resource access across multiple queues.

�ź�����Semaphores�������������ƿ������е���Դ���ʡ�

ʾ���д���������`VkSemaphore`����renderCompleteSemaphore��presentCompleteSemaphore��

```
VK_CHECK_RESULT(swapChain.acquireNextImage(presentCompleteSemaphore, &currentBuffer));
...
VkSubmitInfo submitInfo = {};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
submitInfo.pWaitDstStageMask = &waitStageMask;               // Pointer to the list of pipeline stages that the semaphore waits will occur at
submitInfo.pWaitSemaphores = &presentCompleteSemaphore;      // Semaphore(s) to wait upon before the submitted command buffer starts executing
submitInfo.waitSemaphoreCount = 1;                           // One wait semaphore
submitInfo.pSignalSemaphores = &renderCompleteSemaphore;     // Semaphore(s) to be signaled when command buffers have completed
submitInfo.signalSemaphoreCount = 1;                         // One signal semaphore
submitInfo.pCommandBuffers = &drawCmdBuffers[currentBuffer]; // Command buffers(s) to execute in this batch (submission)
submitInfo.commandBufferCount = 1;                           // One command buffer
VK_CHECK_RESULT(vkQueueSubmit(queue, 1, &submitInfo, waitFences[currentBuffer]));
VkResult present = swapChain.queuePresent(queue, currentBuffer, renderCompleteSemaphore);
```

*`VkSubmitInfo`��`pWaitSemaphores`�ֶ�ע�ͱ������ύ����忪ʼִ��ʱ�ͷ����ź�����*

* Events

> Events provide a fine-grained synchronization primitive which can be signaled either within a command buffer or by the host, and can be waited upon within a command buffer or queried on the host.

�¼���Events���ṩ��һ��ϸ���ȵ�ͬ��ԭ�����������������ڻ������������źţ�Ҳ��������������еȴ����������Ͻ��в�ѯ��

* Pipeline Barriers

> Pipeline barriers also provide synchronization control within a command buffer, but at a single point, rather than with separate signal and wait operations.

�ܵ����ϣ�Pipeline barriers��ͬ������������ṩͬ�����ƣ����ڵ������ϣ������ǵ������źź͵ȴ�������

* Render Passes

> Render passes provide a useful synchronization framework for most rendering tasks, built upon the concepts in this chapter. Many cases that would otherwise need an application to use other synchronization primitives can be expressed more efficiently as part of a render pass.

���ڱ����еĸ����Ⱦ���̣�Render Passes��Ϊ�������Ⱦ�����ṩһ�����õ�ͬ����ܡ������ҪӦ�ó���ʹ������ͬ��ԭ���������Ը���Ч�ر�ʾΪ��Ⱦ���ݵ�һ���֡�

##### 1.6 Vertex Buffer��Index Buffer

ʾ����ʾ�������������ݴ洢��ʽ��host buffer��device buffer��

###### 1.6.1 host buffer

```
// Create host-visible buffers only and use these for rendering. This is not advised and will usually result in lower rendering performance

// Vertex buffer
VkBufferCreateInfo vertexBufferInfo = {};
vertexBufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
vertexBufferInfo.size = vertexBufferSize;
vertexBufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;

// Copy vertex data to a buffer visible to the host
VK_CHECK_RESULT(vkCreateBuffer(device, &vertexBufferInfo, nullptr, &vertices.buffer));
vkGetBufferMemoryRequirements(device, vertices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
// VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT is host visible memory, and VK_MEMORY_PROPERTY_HOST_COHERENT_BIT makes sure writes are directly visible
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &vertices.memory));
VK_CHECK_RESULT(vkMapMemory(device, vertices.memory, 0, memAlloc.allocationSize, 0, &data));
memcpy(data, vertexBuffer.data(), vertexBufferSize);
vkUnmapMemory(device, vertices.memory);
VK_CHECK_RESULT(vkBindBufferMemory(device, vertices.buffer, vertices.memory, 0));

// Index buffer
VkBufferCreateInfo indexbufferInfo = {};
indexbufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
indexbufferInfo.size = indexBufferSize;
indexbufferInfo.usage = VK_BUFFER_USAGE_INDEX_BUFFER_BIT;

// Copy index data to a buffer visible to the host
VK_CHECK_RESULT(vkCreateBuffer(device, &indexbufferInfo, nullptr, &indices.buffer));
vkGetBufferMemoryRequirements(device, indices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &indices.memory));
VK_CHECK_RESULT(vkMapMemory(device, indices.memory, 0, indexBufferSize, 0, &data));
memcpy(data, indexBuffer.data(), indexBufferSize);
vkUnmapMemory(device, indices.memory);
VK_CHECK_RESULT(vkBindBufferMemory(device, indices.buffer, indices.memory, 0));
```

###### 1.6.2 device buffer

```
// Static data like vertex and index buffer should be stored on the device memory
// for optimal (and fastest) access by the GPU
//
// To achieve this we use so-called "staging buffers" :
// - Create a buffer that's visible to the host (and can be mapped)
// - Copy the data to this buffer
// - Create another buffer that's local on the device (VRAM) with the same size
// - Copy the data from the host to the device using a command buffer
// - Delete the host visible (staging) buffer
// - Use the device local buffers for rendering

struct StagingBuffer {
	VkDeviceMemory memory;
	VkBuffer buffer;
};

struct {
	StagingBuffer vertices;
	StagingBuffer indices;
} stagingBuffers;

// Vertex buffer
VkBufferCreateInfo vertexBufferInfo = {};
vertexBufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
vertexBufferInfo.size = vertexBufferSize;
// Buffer is used as the copy source
vertexBufferInfo.usage = VK_BUFFER_USAGE_TRANSFER_SRC_BIT;
// Create a host-visible buffer to copy the vertex data to (staging buffer)
VK_CHECK_RESULT(vkCreateBuffer(device, &vertexBufferInfo, nullptr, &stagingBuffers.vertices.buffer));
vkGetBufferMemoryRequirements(device, stagingBuffers.vertices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
// Request a host visible memory type that can be used to copy our data do
// Also request it to be coherent, so that writes are visible to the GPU right after unmapping the buffer
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &stagingBuffers.vertices.memory));
// Map and copy
VK_CHECK_RESULT(vkMapMemory(device, stagingBuffers.vertices.memory, 0, memAlloc.allocationSize, 0, &data));
memcpy(data, vertexBuffer.data(), vertexBufferSize);
vkUnmapMemory(device, stagingBuffers.vertices.memory);
VK_CHECK_RESULT(vkBindBufferMemory(device, stagingBuffers.vertices.buffer, stagingBuffers.vertices.memory, 0));

// Create a device local buffer to which the (host local) vertex data will be copied and which will be used for rendering
vertexBufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_TRANSFER_DST_BIT;
VK_CHECK_RESULT(vkCreateBuffer(device, &vertexBufferInfo, nullptr, &vertices.buffer));
vkGetBufferMemoryRequirements(device, vertices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &vertices.memory));
VK_CHECK_RESULT(vkBindBufferMemory(device, vertices.buffer, vertices.memory, 0));

// Index buffer
VkBufferCreateInfo indexbufferInfo = {};
indexbufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
indexbufferInfo.size = indexBufferSize;
indexbufferInfo.usage = VK_BUFFER_USAGE_TRANSFER_SRC_BIT;
// Copy index data to a buffer visible to the host (staging buffer)
VK_CHECK_RESULT(vkCreateBuffer(device, &indexbufferInfo, nullptr, &stagingBuffers.indices.buffer));
vkGetBufferMemoryRequirements(device, stagingBuffers.indices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &stagingBuffers.indices.memory));
VK_CHECK_RESULT(vkMapMemory(device, stagingBuffers.indices.memory, 0, indexBufferSize, 0, &data));
memcpy(data, indexBuffer.data(), indexBufferSize);
vkUnmapMemory(device, stagingBuffers.indices.memory);
VK_CHECK_RESULT(vkBindBufferMemory(device, stagingBuffers.indices.buffer, stagingBuffers.indices.memory, 0));

// Create destination buffer with device only visibility
indexbufferInfo.usage = VK_BUFFER_USAGE_INDEX_BUFFER_BIT | VK_BUFFER_USAGE_TRANSFER_DST_BIT;
VK_CHECK_RESULT(vkCreateBuffer(device, &indexbufferInfo, nullptr, &indices.buffer));
vkGetBufferMemoryRequirements(device, indices.buffer, &memReqs);
memAlloc.allocationSize = memReqs.size;
memAlloc.memoryTypeIndex = getMemoryTypeIndex(memReqs.memoryTypeBits, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT);
VK_CHECK_RESULT(vkAllocateMemory(device, &memAlloc, nullptr, &indices.memory));
VK_CHECK_RESULT(vkBindBufferMemory(device, indices.buffer, indices.memory, 0));

// Buffer copies have to be submitted to a queue, so we need a command buffer for them
// Note: Some devices offer a dedicated transfer queue (with only the transfer bit set) that may be faster when doing lots of copies
VkCommandBuffer copyCmd = getCommandBuffer(true);

// Put buffer region copies into command buffer
VkBufferCopy copyRegion = {};

// Vertex buffer
copyRegion.size = vertexBufferSize;
vkCmdCopyBuffer(copyCmd, stagingBuffers.vertices.buffer, vertices.buffer, 1, &copyRegion);
// Index buffer
copyRegion.size = indexBufferSize;
vkCmdCopyBuffer(copyCmd, stagingBuffers.indices.buffer, indices.buffer,	1, &copyRegion);

// Flushing the command buffer will also submit it to the queue and uses a fence to ensure that all commands have been executed before returning
flushCommandBuffer(copyCmd);

// Destroy staging buffers
// Note: Staging buffer must not be deleted before the copies have been submitted and executed
vkDestroyBuffer(device, stagingBuffers.vertices.buffer, nullptr);
vkFreeMemory(device, stagingBuffers.vertices.memory, nullptr);
vkDestroyBuffer(device, stagingBuffers.indices.buffer, nullptr);
vkFreeMemory(device, stagingBuffers.indices.memory, nullptr);
```

##### 1.7


#### 2 Pipelines

> Using pipeline state objects (pso) that bake state information (rasterization states, culling modes, etc.) along with the shaders into a single object, making it easy for an implementation to optimize usage (compared to OpenGL's dynamic state machine). Also demonstrates the use of pipeline derivatives.

Pipelinesʾ����װ��`vkglTF::Model`���`vks::VulkanDevice`�࣬������ܶࡣ

// To do
