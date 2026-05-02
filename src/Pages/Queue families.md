
A **queue family** is a group of GPU queues that support a certain set of operations, described by 

`VkQueueFlags`:  

- **Graphics** (drawing, rendering commands)

- **Compute** (compute shaders)

- **Transfer** (copy/memory operations)

- **Sparse binding**, etc.

But there’s one more that **isn’t covered by `VkQueueFlags`**:  

- **Presentation** — the ability to present images from a [[Swapchain]] to a [[Surface KHR]] (i.e. show them on the window/screen).  

## Present queue

A **present queue** is simply a queue from a queue family that has the ability to **present [[Swapchain]] images to a given [[Surface KHR]]**.  

- Not every graphics or compute queue family can present.
- Whether a queue family can present depends on the **window system integration (WSI)** and must be queried separately.

How you check if a queue family supports present

	vkGetPhysicalDeviceSurfaceSupportKHR(
		physicalDevice,
		queueFamilyIndex,
		surface,
		&supported
	 );

`supported` will be `VK_TRUE` if that queue family supports presentation to the given `surface`.

When creating a logical device and [[Swapchain]], you must ensure you have:  

- At least **one graphics-capable queue** (to render images).
- At least **one present-capable queue** (to present those images to the window).
 
 Sometimes the **same queue family** supports both graphics **and** present → easiest case.
 
 But on some platforms (like some Intel + Windows drivers), the graphics family and the present family are different → then you need to create two queues and handle ownership transfers between them.  

in short:  
	A **present queue** = a queue that can **present [[Swapchain]] images to a window [[Surface KHR]]**.  
  