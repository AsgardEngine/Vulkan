In **Vulkan**, the **swap extent** refers to the **resolution (width × height) of the images in the swapchain** — i.e., the actual pixel dimensions that your rendered images will have before being presented to the screen.


### Where it comes from

- When creating a swapchain (`vkCreateSwapchainKHR`), you must specify the **extent** in `VkSwapchainCreateInfoKHR::imageExtent`.
- This defines how big each swapchain image is (in pixels).

###  How to determine it

The possible values are given by the surface capabilities:  
  
VkSurfaceCapabilitiesKHR capabilities;
vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &capabilities);

`capabilities.currentExtent` → If the window system dictates a fixed size (common on many desktop platforms), you **must** use this value.

Otherwise, Vulkan provides `minImageExtent` and `maxImageExtent`, and you must choose an extent within that range (typically the size of the window’s framebuffer).

### Why it matters:

**Rendering resolution**: All swapchain images will be created with this size, so your render targets and framebuffers must match.

**Window resizing**: If the window changes size, you must recreate the swapchain with a new extent.

**Performance**: Larger extents = more pixels to render, impacting performance.

### Example

If your window is 1280×720 and the surface allows it:  
- Swap extent = (1280, 720)
- All swapchain images will be 1280×720.
- Your framebuffers and viewport/scissor rectangles must also match this size.

   ---

The **swap extent** = the pixel dimensions of each swapchain image (the resolution of your output frame). It is usually equal to the window size, and must be handled carefully on resize events.