Declaring an uniform in shader require the structure holding the variable and a constantbuffer.

```
  struct UniformBuffer 
  {
	  float4x4 model;
	  float4x4 view;
	  float4x4 proj;
  };
  
  [[vk::binding(0, 0)]]
  ConstantBuffer ubo;
```

We also need the same struct on the cpp side with the size of each variable describe throw an alignas.
```
	struct UniformBufferObject 
	{
	  alignas(16) glm::mat4 model;
	  alignas(16) glm::mat4 view;
	  alignas(16) glm::mat4 proj;
	};

```

We now need to allocate memory on the gpu side. For it we need a gpu buffer to describe a gpu region of memory and hwo to use it. We also need a buffer memory that hold the data and bind it to the gpu buffer, and finally we need a mapped buffer that is a pointer to the gpu buffer memory for the cpu to access it.

```
uniformBuffers.clear();
uniformBuffersMemory.clear();
uniformBuffersMapped.clear();

for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++) 
{
	vk::DeviceSize bufferSize = sizeof(UniformBufferObject);
	vk::raii::Buffer buffer({});
	vk::raii::DeviceMemory bufferMem({});
	
	createBuffer(
	  bufferSize,
	  vk::BufferUsageFlagBits::eUniformBuffer,
	  vk::MemoryPropertyFlagBits::eHostVisible | vk::MemoryPropertyFlagBits::eHostCoherent,
	  buffer,
	  bufferMem
	);
	
	uniformBuffers.emplace_back(std::move(buffer));
	uniformBuffersMemory.emplace_back(std::move(bufferMem));
	uniformBuffersMapped.emplace_back( uniformBuffersMemory[i].mapMemory(0, bufferSize));
}

```

 Now we have a way to update the data on the device, we need to connect the shader uniform struct to the gpu memorie dedicated to the uniform. This is done using a descriptor layout. We create a WriteDescriptorSet with the buffer and it's size and tell it's an uniform buffer and tell to which descriptor sets is at destination of (one write descriptor by frame in flight). Also give the position of the uniform binding
 
```
	vk::DescriptorBufferInfo bufferInfo{};
	bufferInfo.buffer = uniformBuffers[i];
	bufferInfo.offset = 0;
	bufferInfo.range = sizeof(UniformBufferObject);
	  
	vk::WriteDescriptorSet bufferDescriptorWrite{};
	bufferDescriptorWrite.dstSet = descriptorSets[i];
	bufferDescriptorWrite.dstBinding = 0;
	bufferDescriptorWrite.dstArrayElement = 0;
	bufferDescriptorWrite.descriptorCount = 1;
	bufferDescriptorWrite.descriptorType = vk::DescriptorType::eUniformBuffer;
	bufferDescriptorWrite.pBufferInfo = &bufferInfo;
	  
	std::array descriptorWrites{bufferDescriptorWrite};
	  
	device.updateDescriptorSets(descriptorWrites, {});	

```

The default set up is done we now take care of what happen at runtime. Starting by updating the uniform.

We set the struct and copy the data to the device buffer memory (of the current image) using the buffer mapped pointer of the current image:

```
	UniformBufferObject ubo{};
	ubo.model = glm::mat4(1.0f);
	
	ubo.view = lookAt(glm::vec3(0.0f, 0.0f, 2.0f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 1.0f, 0.0f));
	ubo.proj = glm::perspective(glm::radians(45.0f), static_cast(swapChainExtent.width) / static_cast(swapChainExtent.height), 0.1f, 10.0f);
	ubo.proj[1][1] *= -1;
	
	memcpy(uniformBuffersMapped[currentImage], &ubo, sizeof(ubo));
    
```

And finally we bind the descriptor of the current frame to the command buffer with the set at 0. This tell the gpu which buffer to use

```
commandBuffers[currentFrame].bindDescriptorSets(vk::PipelineBindPoint::eGraphics, pipelineLayout, 0, *descriptorSets[currentFrame], nullptr);
```

# FURTHER

### Shader declaration

You define a struct in HLSL/Slang and wrap it in a `ConstantBuffer`.  

Using `[[vk::binding(0, 0)]]` is good practice in Vulkan, so you don’t have to “guess” the binding.

```
	struct UniformBuffer 
	{
		float4x4 model;
		float4x4 view;
		float4x4 proj;
	};
	
	[[vk::binding(0, 0)]]
	ConstantBuffer ubo;
```

- **C++ side struct**

  Using `alignas(16)` is important because Vulkan’s **std140 layout** rules require each `mat4` (or `vec4`) to be aligned to 16 bytes.  
  
Using `glm` with `std140` compatible types is a common way to stay safe.

```
	struct UniformBufferObject 
	{
		alignas(16) glm::mat4 model;
		alignas(16) glm::mat4 view;
		alignas(16) glm::mat4 proj;
	};
    
```

### GPU buffer allocation & mapping

`VkBuffer` = describes usage (uniform buffer).
`VkDeviceMemory` = actual allocated memory.
`mapMemory` = gives CPU a pointer to that memory.
 

### Descriptor setup

`VkDescriptorSetLayoutBinding` describes what goes in each binding.

`vk::WriteDescriptorSet` links **buffer → binding slot → descriptor set**.

Matching `dstBinding = 0` with `[[vk::binding(0,0)]]` is exactly how you ensure GPU and shader agree.

### Updating the uniform at runtime

Fill the CPU struct (`UniformBufferObject ubo{}`),

`memcpy` it to the `mapped` pointer of the correct frame’s buffer.

If memory is not `HOST_COHERENT`, you’d also need `flushMappedMemoryRanges`. But since you’re using `HOST_VISIBLE | HOST_COHERENT`, you’re safe. 

### Binding descriptor set at draw time

Binding descriptor set with `pipelineLayout` tells the GPU: “when the shader asks for `[[vk::binding(0,0)]]`, use the UBO bound here.”

Since you have one descriptor set per frame, binding `descriptorSets[currentFrame]` ensures each frame uses its own UBO.

### Resume

- Shader declares what it expects (with set/binding).
- CPU mirrors struct with correct alignment.
- Allocate per-frame buffers, memory, and mapped pointers.
- Hook those up with descriptor sets.
- Update per frame via mapped pointers.
- Bind descriptor sets when recording command buffers.