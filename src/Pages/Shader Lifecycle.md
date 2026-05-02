- We write the shader in slang HLSL and we convert it in SPIR-V the converted file is the file use by the app.
- To hold this binary code in our app we will use a Shader module. We upload the binary code on host memory and create a shader module of the size of the code with the actual code data

```
auto code = readFile(stringPath);
vk::raii::ShaderModule shaderModule = createShaderModule(code);

[[nodiscard]] vk::raii::ShaderModule createShaderModule(const std::vector& code) const 
{
	vk::ShaderModuleCreateInfo createInfo{};
	createInfo.codeSize = code.size() * sizeof(char);
	createInfo.pCode = reinterpret_cast(code.data());
	vk::raii::ShaderModule shaderModule{ device, createInfo };
	
	return shaderModule;
}
```

- Now we need to attach it to the stage of the pipeline the code is made for. We have one script for two stage the vertex and fragment shader. Foreach stage we notify which function the stage need to call

```
vk::PipelineShaderStageCreateInfo vertShaderStageInfo{};
vertShaderStageInfo.stage = vk::ShaderStageFlagBits::eVertex;
vertShaderStageInfo.module = shaderModule;
vertShaderStageInfo.pName = "vertMain";

vk::PipelineShaderStageCreateInfo fragShaderStageInfo{};
fragShaderStageInfo.stage = vk::ShaderStageFlagBits::eFragment;
fragShaderStageInfo.module = shaderModule;
fragShaderStageInfo.pName = "fragMain";

vk::PipelineShaderStageCreateInfo shaderStages[] = {vertShaderStageInfo, fragShaderStageInfo};
```

- At the end we group both stage and add it to the graphic pipeline. At this point vulkan read/compile/resolve/optimize and store the result. Where the fuck it's store the result? host? device? vulkan abstract it so we will never know...

```
 vk::GraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.pNext = &pipelineRenderingCreateInfo;
pipelineInfo.stageCount = 2;
pipelineInfo.pStages = shaderStages;
...
graphicsPipeline = vk::raii::Pipeline(device, nullptr, pipelineInfo);
```

- On draw frame we will bind the pipeline to the command buffer   

```
commandBuffers[currentFrame].bindPipeline(vk::PipelineBindPoint::eGraphics, *graphicsPipeline);
```

### FURTHER

- Your summary of the **lifecycle of a basic shader in Vulkan** is pretty solid! 👍 It captures the main steps from writing the shader source to binding the pipeline at draw time. Let me give you some feedback and refinements to make it more precise and complete:


**SPIR-V input alignment**

- The Vulkan spec requires `pCode` to be a pointer to an array of 32-bit words (`uint32_t`), and `codeSize` must be a multiple of 4.

**ShaderModule lifetime**

- The `ShaderModule` is only needed during **pipeline creation**. Once the pipeline object is created, the shader module can be destroyed.
- Keeping it around during rendering isn’t necessary.

**Multiple shaders in one SPIR-V**

- If both are in one file, the `pName` fields select the correct entry points, but many tools/pipelines assume one module per stage.

**Pipeline creation context**

- A full `GraphicsPipelineCreateInfo` usually includes info about input assembly, viewport state, rasterizer, multisampling, color blending, etc.

**Binding**

- To issue draw calls, the pipeline must be bound. Often you’ll also bind descriptor sets and vertex buffers at this stage.

### Lifecycle Summary
- **Write shaders** (HLSL, GLSL, etc.) → compile to SPIR-V.
- **Load SPIR-V** into the app and create a `vk::ShaderModule`.
- **Set up shader stage info** (`vk::PipelineShaderStageCreateInfo`) with stage flag + entry point.
- **Create a graphics pipeline**, passing in all stages + fixed-function state.
- **Destroy ShaderModules** after pipeline creation (optional).
- **During rendering**, bind pipeline + descriptor sets + buffers, then issue draw commands.

---

 the shader module (`vk::ShaderModule`) is only needed during **pipeline creation** (`vkCreateGraphicsPipelines` / `vk::raii::Pipeline`).  
  
At that point, the driver consumes the SPIR-V code, performs validation, compilation to GPU machine code (implementation-dependent), and copies what it needs internally.  
  
After the pipeline object is created successfully, you can **destroy the shader module immediately** — the pipeline no longer depends on it.  

 ### Detailed Explanation
- **When you create the shader module (`vkCreateShaderModule`)**
	
	- Vulkan does **not** compile or upload the code to the GPU yet.
	- It just validates the SPIR-V binary format and stores it in a lightweight handle.

- **When you create the pipeline (`vkCreateGraphicsPipelines`)**
	
	- This is the step where Vulkan:
		
		- reads the SPIR-V from the attached shader modules,
		- compiles/translates it to the GPU’s native ISA (machine code),
		- resolves specialization constants,
		- optimizes and links stages together,
		- and stores the result inside the pipeline object.
	
	- At this point, the actual code/data is copied into **driver-managed memory** (device or host, depending on implementation).
	
- **After pipeline creation**
	
	- The pipeline has all the executable code it needs.
	- The shader module handle is no longer required and can be destroyed.
	- Keeping it around does not affect execution (but wastes memory).


-  Best Practice
    
      
    From the [Vulkan spec](https://registry.khronos.org/vulkan/specs/1.3-extensions/html/chap9.html#pipelines-shaders):  
      
    _“Once a pipeline has been created, the application may destroy the shader modules used in its creation, without affecting the pipeline.”_  


### FURTHER MEMORY

- That’s a subtle but very important point. In Vulkan, **you don’t directly see or control where the driver places the final compiled GPU ISA (machine code)** — whether it lives in device-local memory, host-visible memory, or some internal opaque storage.


What actually happens
- **SPIR-V stage**

	- When you call `vkCreateShaderModule`, the driver just validates and stores the SPIR-V bytecode.
	- No GPU ISA is generated yet, and no device memory is allocated.

- **Pipeline creation (`vkCreateGraphicsPipelines`)**

	- The driver consumes the SPIR-V, compiles it to GPU machine code (ISA), and places it in an internal memory region.
	- Where it goes (host memory, device-local VRAM, or some hybrid) is entirely up to the driver and hardware.
	- This is why shader module handles are no longer needed afterward.

- **Execution time**

	- The GPU executes from that internal ISA representation.
	- The app has no API to query or control its location.
    


Can you “see” where it’s stored?
- **No direct way in Vulkan API**: Vulkan deliberately abstracts this away. You cannot ask: _“Did the ISA land in device memory or host memory?”_
- **Indirect clues**:

- Some drivers expose shader cache behavior (pipeline cache objects, e.g., via `VkPipelineCache`), which may be backed by host memory.
- Tools like **RenderDoc**, **Nsight**, or **Radeon GPU Analyzer** let you inspect the compiled ISA, but this is outside Vulkan itself.

- **Vendor extensions**: A few extensions (like `VK_AMD_shader_info`, `VK_KHR_pipeline_executable_properties`) can query statistics about compiled shaders (e.g., number of SGPRs, code size, disassembly), but still don’t let you control memory residency.

---
- **Step the ISA is generated** → at **pipeline creation** (`vkCreateGraphicsPipelines`).
- **Step it’s uploaded to device/host memory** → opaque, handled by the driver at that same time.
- **Control/visibility** → you cannot dictate or query the actual memory placement through core Vulkan.
