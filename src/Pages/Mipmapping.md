In **Vulkan (and graphics in general)**, **mipmapping levels** are a sequence of pre-computed, progressively smaller versions of a texture.

What mipmaps are:
- Level **0**: the full-resolution texture (e.g., 1024×1024).
- Level **1**: half the size (512×512).
- Level **2**: half again (256×256).
- … until you reach 1×1.

Each downsampled image is called a **mipmap level**.  

**Performance**:

- Sampling from smaller textures reduces memory bandwidth and improves cache usage when an object appears small on screen.

**Image quality**:

- Prevents aliasing and shimmering when textures are viewed at a distance or at sharp angles.

# How it works:

The GPU automatically chooses which mipmap level to sample from based on the **screen-space size of the texture** being rendered.
 
You can control filtering via sampler settings (`VkSamplerCreateInfo::mipmapMode`).

- `VK_SAMPLER_MIPMAP_MODE_NEAREST` → pick the nearest level.
- `VK_SAMPLER_MIPMAP_MODE_LINEAR` → interpolate smoothly between levels.


# Vulkan details:

When creating an image (`VkImageCreateInfo::mipLevels`), you specify how many mip levels it has.

To use mipmaps, you need to:

- Allocate space for all mip levels when creating the image.
- Fill/generate each level (either offline, or at runtime using `vkCmdBlitImage` or compute shaders).
- Use a `VkImageView` that includes the desired mip levels.
- Create a sampler with mipmapping enabled.


# Example
  
If you create a **1024×1024 texture**:  

- Level 0: 1024×1024
- Level 1: 512×512
- Level 2: 256×256
- …
- Level 10: 1×1
  
That makes **11 mip levels** (`floor(log2(max(width, height))) + 1`).  

---

**Mipmapping levels** are a chain of progressively smaller textures stored together, allowing the GPU to sample the most appropriate resolution for performance and visual quality.