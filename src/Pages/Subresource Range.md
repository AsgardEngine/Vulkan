### What is `VkImageSubresourceRange` ?

`VkImageSubresourceRange` is a structure in Vulkan that **defines which parts (“subresources”) of an image you want to operate on**.  
  
An image in Vulkan can have:  

- **Aspects** → e.g., color, depth, stencil, or multiple planes in a multi-planar image.
- **Mipmap levels** → the chain of smaller textures.
- **Array layers** → for texture arrays or cubemaps.
    
    Since you rarely want to operate on the **entire image at once**, `VkImageSubresourceRange` lets you specify exactly which **aspects, mip levels, and array layers** you’re targeting.  

### Where it is used:

- **Image Views (`VkImageViewCreateInfo::subresourceRange`)**

- Defines what part of an image the view will expose.
- Example: A view that only looks at the **depth aspect** of a depth-stencil image.

- **Image Layout Transitions & Barriers (`VkImageMemoryBarrier::subresourceRange`)**

- Defines which parts of an image the barrier applies to.
- Example: Transition **only mip level 0** to `COLOR_ATTACHMENT_OPTIMAL`, leaving other mip levels untouched.

### Fields
```
  typedef struct VkImageSubresourceRange {
	VkImageAspectFlags aspectMask;  // COLOR, DEPTH, STENCIL, etc.
	uint32_t baseMipLevel;          // First mip level to use
	uint32_t levelCount;            // Number of mip levels
	uint32_t baseArrayLayer;        // First array layer to use
	uint32_t layerCount;            // Number of layers
  } VkImageSubresourceRange;
    
```

- **aspectMask** → Which aspect(s) (e.g., `VK_IMAGE_ASPECT_COLOR_BIT`, `VK_IMAGE_ASPECT_DEPTH_BIT`).

- **baseMipLevel / levelCount** → Range of mip levels.

- **baseArrayLayer / layerCount** → Range of array layers.


### Examples

- **Image View for a 2D texture (no mipmaps):**
```
    
      subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
      subresourceRange.baseMipLevel = 0;
      subresourceRange.levelCount = 1;
      subresourceRange.baseArrayLayer = 0;
      subresourceRange.layerCount = 1;
    
```

- **Cubemap View (6 faces):**
```
    
      subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
      subresourceRange.baseMipLevel = 0;
      subresourceRange.levelCount = 1;
      subresourceRange.baseArrayLayer = 0;
      subresourceRange.layerCount = 6; // one for each face
    
```

- **Barrier affecting _all mip levels_ of a depth-stencil image:**
    ```
      subresourceRange.aspectMask = VK_IMAGE_ASPECT_DEPTH_BIT | VK_IMAGE_ASPECT_STENCIL_BIT;
      subresourceRange.baseMipLevel = 0;
      subresourceRange.levelCount = VK_REMAINING_MIP_LEVELS;   // special constant
      subresourceRange.baseArrayLayer = 0;
      subresourceRange.layerCount = VK_REMAINING_ARRAY_LAYERS; // special constant
    
    ```

  ---
      
    `subresourceRange` tells Vulkan **which slices of an image (aspect, mip levels, layers)** are being referenced by an image view or affected by a barrier/transition. Without it, Vulkan wouldn’t know _which part_ of a potentially complex image you mean.  
      
