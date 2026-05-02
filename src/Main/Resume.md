Vulkan Workflow: From Window to Rendering

Vulkan is a low-level graphics and compute API that provides explicit control over the GPU. Unlike older APIs, it requires developers to manage almost every detail of the rendering process. The following is a step-by-step overview of the typical Vulkan workflow for building a simple application that renders to the screen.  

# 1. Creating a Window

Before using Vulkan, we need a windowing system to present images. Libraries like **GLFW** or **SDL** simplify this step by creating a window and handling input. They also provide Vulkan-compatible window [[Surface KHR|surfaces]], which **act as the bridge** between Vulkan and the operating system’s window manager.  

# 2. Initializing Vulkan
 
- Instance
	
	The Vulkan instance is the root of every application. When creating it, you provide metadata (application name, version, engine name, API version) along with two important optional features:  

- [[Layers]] 
	
	Add extra functionality, most commonly [[Validation Layers|validation layers]] for debugging and error checking.

- [[Extensions]] 
	
	Enable optional Vulkan features, such as presenting images to a window [[Surface KHR|surface]].

- [[Validation Layers]]
    
	Intercept API calls and help detect errors or misuse of the Vulkan API. They  can log warnings, errors, and performance hints, making them essential for development.  
    
- [[Surface KHR]]
    
    Represents the area of the screen where [[Image View|Vulkan images]] will be displayed. This object connects Vulkan with the OS window created earlier.  


# 3. Selecting a Physical Device

Next, you select the GPU that will run the application.

The chosen device must support the required Vulkan version, the **[[Swapchain]] extension** for presentation, and appropriate **[[Queue families]]** for graphics operations. Dedicated GPUs are usually preferred, but integrated GPUs may also work for simpler use cases.  

# 4. Creating a Logical Device

Once a **physical** device is selected, a **logical** device is created to serve as the application’s handle to the GPU. Here, you specify which [[Surfaces Capabilities|surfaces capabilities]], [[Extensions|extensions]], and [[Queue families|queue families]] you want to use. The logical device is what the application uses to submit [[Commands|commands]] and manage resources.  

# 5. The [[Swapchain]]

Is a collection of [[Image View|images]] that are presented to the window in sequence. Typically, two or three images are used to enable double or triple buffering. When creating the swapchain, you define:  

- The image format (color format, depth format).

- The resolution (matching the window’s surface).

- The number of images.
	
	Each swapchain image is then associated with **image views**, which describe how the GPU should interpret and use the image.  


# 6. Buffers and Resources

- [[Vertex Buffer]] and [[Index Buffer]]
	
	Vertices define the geometry, while index buffers avoid duplication by reusing vertices. Data is usually first placed in a staging buffer (CPU-visible memory) and then copied to GPU memory for efficient access during rendering.  

- [[Uniform Buffer]]
    
	Uniform buffers store data that remains constant across many shader invocations, such as transformation matrices. One uniform buffer per frame is typically required to allow updates without stalling the GPU.  

- [[Descriptors]]
	  
    Descriptors tell shaders where to find resources like buffers, textures, and samplers. They are organized into descriptor sets, which are bound to the pipeline at draw time.

# 7. [[Graphics Pipeline]]

The graphics pipeline defines every step from input data to final pixels on the screen. It is a fixed, immutable object that must be created in advance. Key components include:  

- **[[Shader Module]]** – Contain the compiled vertex and fragment shaders.

- **Vertex Input** – Describes how vertex data is laid out and connected to shader inputs.

- [[Input Assembler]] – Defines how vertices are grouped into primitives (e.g., triangles).

- **[[Scissor State]]** – Control the area of the framebuffer that will be rendered to.

- [[Rasterization]] – Converts primitives into fragments.

- [[Multisampling]] – Improves image quality by reducing aliasing.

- **[[Color Blending]]** – Determines how new fragments blend with existing framebuffer data.

- **Pipeline Layout** – Connects [[Descriptor|descriptor sets]] and push constants to the pipeline.

The pipeline is then linked with the swapchain render targets to produce visible images.  

# 8. Command Management

[[Commands]]:
	Commands in Vulkan are recorded into **command buffers**, which are submitted to GPU queues for execution. Command pools manage the memory for these command buffers.  

Recording Commands  
	Typical recorded commands include binding pipelines, setting viewports and scissors, binding vertex/index buffers, binding descriptor sets, and issuing draw calls.  


# 9. Synchronization

Vulkan requires explicit synchronization between the CPU and GPU. The two main mechanisms are:  

- [[Semaphores]] – Used for GPU-to-GPU synchronization, such as ensuring an image is ready before rendering starts.

- **[[Fences]]** – Used for CPU-to-GPU synchronization, allowing the CPU to wait until the GPU finishes a task.

Proper synchronization ensures frames are rendered and presented in the correct order.  

# 10. Drawing a Frame

With everything initialized, rendering begins:  

- Wait for the fence from the previous frame.
- Acquire the next swapchain image.
- Update uniform buffers with per-frame data.
- Reset and record a command buffer for this frame.
- Submit the command buffer to the graphics queue, signaling semaphores.
- Present the image to the presentation queue.
- If the swapchain is invalid (e.g., after a window resize), recreate it along with dependent resources.
 
This process repeats every frame, producing smooth animation.  

# Conclusion

Vulkan requires a significant amount of setup before the first pixel is drawn. From creating a window and instance, selecting a device, building pipelines, and managing synchronization, every step gives developers fine-grained control over performance and rendering. Although complex, this workflow provides the foundation for highly efficient, cross-platform graphics applications.  

# Texture

- createTexture image after command pool
- We start by loading the image using the std_load lib in cpu only memory, we create a staging buffer to receive the image data, we transfer the data to the staging buffer, we also create a Vk image (For the buffer to know how to receive the texture image) and a buffer in gpu only memory, then we copy the staging buffer data to the gpu-only buffer memory.

- We now need a texture image view with an image sampler
- We need to refind the descriptor to acknowledge the shader of the image texture
- Modifying vertex struct to add uv coordinate and shader to display the texture

# Depth buffering

- Add the depth stencil state to the graphic pipeline
- And set the format to the depth attachement for rendering
- We need to create one vk image to store the depth attachmen and one image view to look at the image the image is directly created gpu onmy memory don't need for staging buffer we have no data to transfer from cpu memory to gpu
- In recordCommandBuffer transition the depth image in better suitable layout for the gpu to handle
- Set the rendering attachment info, don't forget to set the storeOp to eDontCare because the depth is only use to discard not visible fragment when processing color attachment their is no interest to kepp the image and in the case we will store the image we should store it in different swapchain to keep the color attachment and depth attachment.
- If recreate swapchain also recreate depthRespurce to handle the new format size etc...