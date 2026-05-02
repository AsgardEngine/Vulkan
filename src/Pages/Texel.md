In **Vulkan terminology**, a **texel** (short for _texture element_) is essentially the **pixel of a texture** — the smallest addressable unit of a texture image.

More precisely:
- A **pixel** is the smallest unit of a rendered image on the screen.
- A **texel** is the smallest unit of a stored texture in memory.
	
	When a shader samples a texture, it reads **texels** from the GPU memory and applies filtering (nearest, linear, mipmaps, anisotropy, etc.) to compute the final **pixel color** that ends up on screen.  
	

In Vulkan:
- Texels live in a **`VkImage`** (2D, 3D, or cube map).
- A texel’s data format is defined by the **image format** (e.g., `VK_FORMAT_R8G8B8A8_SRGB` = 4 channels, 8 bits each).
- When you bind an image as a sampled texture, the **sampler** fetches texels, possibly blending multiple ones depending on filtering mode.
- Texels can be accessed in shaders using functions like `texture()`, `texelFetch()`, or `imageLoad()`.

## Example

- A 512 × 512 texture has
    
    `512 × 512 = 262,144 texels`.  
    
- If the format is `R8G8B8A8_UNORM`, each texel = 4 bytes (1 byte per channel).
- So the whole texture = `262,144 × 4 = 1,048,576 bytes` (1 MB).
