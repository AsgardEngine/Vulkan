
In **Vulkan**, a **swapchain** is the mechanism that manages the series of images that will be presented to the screen (usually through the windowing system).

It's a **queue of images (framebuffers)** that the GPU renders into, and the display system takes from, to show on the monitor.  

# What is a Swapchain

 A **swapchain** is a collection of **[[VkImage]]** objects that represent the images shown on the screen.

 It is created from the `VK_KHR_swapchain` [[Extensions]], which is essential for most Vulkan applications that display graphics.

 You don't render directly to the screen; instead, you render to one of the swapchain images, then present it.

# Purpose of a Swapchain

- **Synchronize Rendering and Display**
	
	Prevents tearing and ensures smooth presentation by controlling when images are displayed.
	Supports different presentation modes (e.g., vsync, mailbox, immediate).

- **Buffering**
	
	Usually uses **double buffering** (2 images) or **triple buffering** (3 images).
	While one image is being displayed, another can be rendered into.
	Improves smoothness and reduces latency.

- **Format and Resolution Control**
	
	The swapchain defines image format (color depth, sRGB), resolution, and how many images are in the chain.

- **Efficient Presentation**
	
	Interfaces directly with the window system (e.g., Windows, X11, Wayland, macOS MoltenVK).

# Swapchain Workflow

- **Acquire an image** from the swapchain (`vkAcquireNextImageKHR`).

- **Render** into that image using command buffers.

- **Present** the image to the screen (`vkQueuePresentKHR`).

# Presentation Modes

- **FIFO (vsync, always available)**
    
	 Display waits for vertical blank; prevents tearing.  

- **Mailbox**
    
    Low-latency triple-buffering; replaces old frames if rendering is faster than display.  

- **Immediate**
    
	 Presents immediately; may cause tearing, lowest latency.  
  
# Summary

A **swapchain** in Vulkan is a series of images used for presenting rendered frames to the screen. It enables smooth rendering through buffering, synchronizes with the display refresh, and defines how images are presented (vsync, tearing, etc.). It is essential for any Vulkan application that outputs to a window.

# Recreation

In **Vulkan**, the **swapchain** is the queue of images that are presented to the screen. Because it is tightly tied to the **surface** (e.g., the window, display resolution, and presentation mode), there are situations where it becomes **invalid** or **suboptimal**, and in those cases you must **recreate the swapchain**.

When should you recreate the swapchain?

- **Window is resized**
	
	- If the framebuffer size changes (user resizes the window, toggles fullscreen, changes resolution), the old swapchain images no longer match the new size.
	- Vulkan functions will return `VK_ERROR_OUT_OF_DATE_KHR`.
	
- **Surface becomes incompatible**
	
	- For example, if the window is minimized, moved to a monitor with a different refresh rate, or the GPU/display settings change.
	- Vulkan may return `VK_SUBOPTIMAL_KHR` or `VK_ERROR_OUT_OF_DATE_KHR`.
	
- **Presentation mode or format must change**
	
	- If you want to switch from `VK_PRESENT_MODE_FIFO_KHR` (vsync) to `VK_PRESENT_MODE_MAILBOX_KHR` (triple-buffered, lower latency).
	- Or if the color format/depth/stencil format must change due to new requirements.
	
- **Swapchain no longer supports required image count**
	
	- If you request more images than the old swapchain supports, you need to recreate it.


How do you know?

On **`vkAcquireNextImageKHR`** or **`vkQueuePresentKHR`** you may get:

- `VK_ERROR_OUT_OF_DATE_KHR` → swapchain must be recreated.
- `VK_SUBOPTIMAL_KHR` → swapchain still works, but should be recreated soon for best results.


Typical recreation steps
- Wait until the device is idle (`vkDeviceWaitIdle`) or synchronize properly so no swapchain images are still in use.
	
- Destroy old swapchain resources:
	
	- Framebuffers
	- Image views
	- The swapchain itself (`vkDestroySwapchainKHR`)
	
- Query surface capabilities again (`vkGetPhysicalDeviceSurfaceCapabilitiesKHR`).
	
- Create a new swapchain with updated parameters (`vkCreateSwapchainKHR`).
	
- Recreate framebuffers, pipelines (if dependent on swapchain dimensions), etc.

---

Recreate the swapchain when it becomes **out of date** (window resize, surface changes, etc.) or **suboptimal** (works but not ideal). This ensures the swapchain images match the current surface properties and can be presented correctly.  


# Swapchain & Frame Buffer

Flow
- **Pipeline + render pass (or dynamic rendering)**
	
	- The pipeline defines which **fragment shader outputs** exist (e.g. `layout(location=0) out vec4 outColor;`).
	- The render pass/dynamic rendering specifies _where those outputs go_ (which attachment index maps to which image).
	
- **Framebuffer**
	
	- A framebuffer is a collection of attachment image views (color, depth, stencil, resolve).
	- In the classic model, you usually make **one framebuffer per swapchain image** (so the right swapchain image is bound as the color target).
	
- **Acquire a swapchain image**
	
	- You call `vkAcquireNextImageKHR` to get which swapchain image is “next” to render into.
	
- **Render into that image**
	
	- You bind the framebuffer (or in dynamic rendering, you specify the attachments inline).
	- Fragment shader outputs are written into those attachments — usually a swapchain image for color, plus maybe a depth image.
	
- **Present the image**
	
	- You submit the command buffer and call `vkQueuePresentKHR`.
	- Vulkan hands the rendered swapchain image back to the **WSI** (window system integration).
	- The OS/compositor takes care of showing it on screen at the right time (vsync, composition, etc.).


---
  
The pipeline defines the fragment shader outputs and how they map to attachments.  
  
Those attachments are part of a framebuffer, and typically we create one framebuffer for each swapchain image.  
  
We acquire a swapchain image, render into it using the framebuffer attachments, and then present it.  
  
The WSI takes care of displaying the presented image on the window/surface.  
