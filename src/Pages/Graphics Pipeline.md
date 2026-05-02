What the graphics pipeline _is_:
	The **graphics pipeline** (a `VkPipeline` created with `VkGraphicsPipelineCreateInfo`) is a single, immutable object that describes the entire sequence of operations the GPU uses to turn your vertex data into pixels on the screen. It packages together both the **programmable shader stages** and the **fixed-function state** into one thing the GPU can optimize for.  
    

 Why it exists / what it’s for:
- **Tell the GPU everything it needs** to render a primitive: how vertices are read, how shaders execute, how primitives are assembled and rasterized, how blending and depth testing are done, etc.
- **Enable heavy driver/hardware optimizations** by giving the GPU a complete, unambiguous description up front.
- **Reduce runtime state changes**: instead of setting many independent states constantly, you bind a pipeline that already encodes those states.

Main parts (high level):
- **Programmable shader stages**
	- Vertex, (optional) Tessellation control/eval, (optional) Geometry, Fragment.
	- Supplied as SPIR-V shader modules.

- **Fixed-function stages / pipeline state**
	- Vertex input (bindings, attribute formats)
	- Input assembly (primitive type, restart)
	- Tessellation state (patch control points) if used
	- Viewport & scissor (can be dynamic)
	- Rasterization state (culling, polygon mode, front face)
	- Multisample state
	- Depth/stencil state (depth test/write, stencil ops)
	- Color blend state (per-attachment blending)
	- Dynamic state list (which of the above can be changed at draw time)

- **Pipeline layout**
	- `VkPipelineLayout` describes descriptor set layouts and push constant ranges the shaders expect.

- **Render pass / subpass compatibility**
	- A graphics pipeline is created for a specific render pass (and subpass index) or must be compatible with it — tells the GPU what attachments/formats will be used.

 How it’s used in a frame:
- Create `VkPipelineLayout` (descriptor set layouts + push constants).
- Create shader modules (SPIR-V) and fill `VkPipelineShaderStageCreateInfo`.
- Fill `VkGraphicsPipelineCreateInfo` with all fixed-function structs + shader stages + pipeline layout + render pass/subpass.
- Create the pipeline (`vkCreateGraphicsPipelines`).
- During a command buffer: `vkCmdBindPipeline(cmd, VK_PIPELINE_BIND_POINT_GRAPHICS, pipeline)`.
- Bind descriptor sets and vertex/index buffers, then `vkCmdDraw` / `vkCmdDrawIndexed`.

Important practical notes:
- **Pipelines are (relatively) expensive to create.** Create them up front (or use background threads) and reuse them.
- **They are effectively immutable.** To change state encoded in the pipeline you generally need a different pipeline — except for things you marked as _dynamic_ (viewport, scissor, line width, etc.).
- **Use `VkPipelineCache`** to speed up creation across runs and potentially share optimizations between runs.
- **Pipeline derivatives & pipeline libraries** exist for faster creation/less duplication (advanced usage).
- **Keep state changes cheap at runtime**: minimize binding different pipelines every draw if possible (batch objects that use the same pipeline).

Quick mapping (stage → purpose):
- **Vertex Input**: how vertex buffer memory maps to shader input attributes.
- **Vertex Shader**: transform vertices, compute per-vertex outputs.
- **Tessellation** (optional): subdivide patches into primitives.
- **Geometry Shader** (optional): generate new geometry on the fly.
- **Rasterization**: convert primitives to fragments (culling, polygon mode).
- **Fragment Shader**: compute final fragment color(s).
- **Per-attachment Color Blend**: combine fragment output with framebuffer contents.
- **Depth & Stencil**: depth testing/writing and stencil operations.
- **Multisampling**: anti-aliasing behavior.

Example: when you need multiple pipelines
- Different shader programs → different pipelines.
- Different blending/depth state (opaque vs additive) → often different pipelines.
- Different vertex formats (position-only vs position+normal+uv) → different vertex input state → different pipeline.


      
    A Vulkan **graphics pipeline** is the compiled, fully-specified description of how the GPU converts your vertex data and shader logic into rasterized fragments and final pixels — combining shaders + all relevant fixed-function state into an optimized object you bind before drawing.