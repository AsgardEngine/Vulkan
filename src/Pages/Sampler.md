- In **Vulkan**, a **sampler** is an object that defines **how an image is accessed in shaders**, specifically how [[Texel]] are read when performing texture sampling operations.
    
      
    It doesn’t hold the image data itself (that’s in an `VkImage`/`VkImageView`), but instead describes **the rules for fetching texels**.  


Here are the main purposes of a sampler:  


- 1. Filtering
      
	When a shader samples a texture, the requested texture coordinates may not map exactly to a single texel. The sampler defines how to handle that:  
    
	- **Nearest filtering** → Pick the nearest texel.
	- **Linear filtering** → Interpolate between surrounding texels.
	- **Anisotropic filtering** → Improve quality when viewing textures at oblique angles.
    
-  2. Addressing Modes
    
    Samplers specify what happens when texture coordinates are **outside [0,1]** range:  
    
	- **Repeat** → Wrap around (tiled textures).
	- **Mirrored repeat** → Flip at each integer boundary.
	- **Clamp to edge** → Stick to edge texel.
	- **Clamp to border** → Use a constant border color.
    
-  3. Mipmapping
    
    If mipmaps are available, the sampler defines:  
    
	- **How to pick mip levels** (nearest, linear, anisotropic).
	- **How to calculate LOD (Level of Detail)** bias or clamp.
    
-  4. Comparison Sampling (Shadow Maps)
    
    A sampler can perform **depth comparisons** instead of returning raw texel values.  
      
    Useful for shadow mapping, where you compare a sampled depth value against a reference.  
      
-  5. Decoupling State from Images
    
    Unlike OpenGL, Vulkan separates:  
    
	- **Image (data storage)** → `VkImage` and `VkImageView`
	- **Sampling rules** → `VkSampler`
    
      
    This makes things more explicit, avoids hidden state, and allows reuse:  
    
	- One sampler can be used with many images.
	- One image can be sampled with different samplers.
    
    ---
      
    A **Vulkan sampler** is like a **"recipe" for reading textures** — it tells the GPU _how_ to look up texels (filtering, wrapping, mipmaps, etc.), while the image object provides the _actual texel data_.  
