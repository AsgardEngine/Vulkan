
- The **“swap format”** (often called **surface format**) is about **how pixels are stored in the images of your swapchain**.


### What it is
  
Each swapchain image has a specific **format** (color encoding) and **color space**.  
  
This is described by a `VkSurfaceFormatKHR` struct:  
  
```
  typedef struct VkSurfaceFormatKHR 
  {
	VkFormat     format;      // e.g. VK_FORMAT_B8G8R8A8_SRGB
	VkColorSpaceKHR colorSpace; // e.g. VK_COLOR_SPACE_SRGB_NONLINEAR_KHR
  } VkSurfaceFormatKHR;
    
```

- **`format`** → Describes the memory layout of pixels in the swapchain image.
	   
	Example: `VK_FORMAT_B8G8R8A8_SRGB` means each pixel has 8 bits for Blue, Green, Red, and Alpha, in sRGB encoding.  

- **`colorSpace`** → Defines how colors are interpreted/displayed.
    
    Example: `VK_COLOR_SPACE_SRGB_NONLINEAR_KHR` is the most common, matching how monitors expect color values.  
  
### Why it matters

When creating the swapchain, you **must pick a surface format supported by both the GPU and the window system**.  
  
Different platforms/GPUs might support different sets of formats, so Vulkan makes you query them first.  
  
That’s why we do:    
```
  swapChainImageFormat = chooseSwapSurfaceFormat(
	physicalDevice.getSurfaceFormatsKHR(surface)
  );
```

- `getSurfaceFormatsKHR(surface)` → asks Vulkan: _“Which (format, colorSpace) pairs can this GPU + OS surface actually use?”_

- `chooseSwapSurfaceFormat(...)` → picks the “best” one, usually preferring something like:

	- `VK_FORMAT_B8G8R8A8_SRGB`
	- with `VK_COLOR_SPACE_SRGB_NONLINEAR_KHR`


### Example “choose” function

A common implementation looks like this:  
      
```
  VkSurfaceFormatKHR chooseSwapSurfaceFormat(
	const std::vector& availableFormats) 
  {
	for (const auto& availableFormat : availableFormats) {
		if (availableFormat.format == VK_FORMAT_B8G8R8A8_SRGB &&
			availableFormat.colorSpace == VK_COLOR_SPACE_SRGB_NONLINEAR_KHR) {
			return availableFormat;
		}
	}
	// If preferred format not found, just return the first available
	return availableFormats[0];
  }
```
   ---

The **swap format** = the **pixel format + color space** used by the swapchain images.  

It ensures the GPU writes pixels in a format the monitor/windowing system understands.  
