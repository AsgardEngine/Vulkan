- **What a Framebuffer Is in Vulkan**
	- In Vulkan, a **Framebuffer** (`VkFramebuffer`) is a **collection of image attachments** (color, depth, stencil, etc.) that the pipeline can **render into during a render pass**.
	- You can think of it as the **destination surfaces** where your rendering results end up.
    
      
    It’s not a pipeline stage like rasterization or fragment shading — it’s a **rendering target object** you configure and bind.  
      
    
    ---
    
- **Main Purposes of the Framebuffer**
	- **Render Target Binding**
		
		- When you begin a **render pass**, you tell Vulkan _which framebuffer_ to use.
		- This defines exactly which images (textures or swapchain images) are written to.
		
	- **Holds Attachments**
		
		- A framebuffer contains **attachments** such as:
			
			- **Color attachments** → where fragment shader outputs final colors.
			- **Depth/stencil attachments** → used for depth and stencil testing.
			- (optionally) resolve attachments (for multisampling).
			
	- **Swapchain Presentation**
		
		- When rendering to the screen, your framebuffer’s **color attachment** is typically one of the **swapchain images** (the ones that get presented to the window).
		
	- **Offscreen Rendering**
		
		- You can create framebuffers from offscreen textures to render things like:
			
			- Shadow maps
			- Post-processing passes (bloom, blur, HDR, etc.)
			- G-buffers (deferred rendering).
	
    ---
    

- **How It Fits in Vulkan’s Flow**
	- You create a **Render Pass** → describes _what kinds of attachments_ (color, depth, etc.) exist and how they’re used.
	- You create a **Framebuffer** → provides the _actual images_ to back those attachments.
	- When you call `vkCmdBeginRenderPass()`, you specify the framebuffer to use → that’s where all your drawing goes until `vkCmdEndRenderPass()`.
	
---
    
- **Analogy**
      
    Think of the framebuffer as the **canvas** you’re painting onto:  
    
	- The **render pass** defines the _rules_ of how to paint (clear before painting, keep old colors, etc.).
	- The **framebuffer** is the actual _piece of paper or canvas_ where the paint (fragments) lands.
	- The **color/depth attachments** are like the different layers of the canvas (color layer, depth layer, stencil layer).
    
    ---
      
    In Vulkan, a **Framebuffer is the set of images (attachments) that a render pass will draw into**. It serves as the final target for rendering operations, whether that’s the screen (via the swapchain) or an offscreen texture.  
      
    
    ---