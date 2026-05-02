
**Multisampling (MSAA – Multisample Anti-Aliasing)**, which is a **rasterization feature** in Vulkan.

**What Multisampling Is**:

Multisampling is a technique used to **reduce aliasing** (jagged edges on diagonal lines or polygon borders).

Instead of shading each pixel once at its center, multisampling takes **multiple coverage samples per pixel** when rasterizing geometry.

These samples determine how much of a pixel is covered by a primitive, producing smoother edges.


**How It Works in Vulkan**:

- **Coverage Sampling**
	
	Each pixel has multiple sub-samples (e.g., 2×, 4×, 8×).
	
	Rasterization checks which sub-samples fall inside the triangle.

- **Single Shading per Pixel**
	
	Unlike supersampling, the **fragment shader usually runs once per pixel**, _not_ once per sample (more efficient).
	
	The result is then applied to all covered sub-samples.

- **Resolve Step**
	
	After rendering into a **multisampled image**, you resolve it into a normal (single-sample) image before presentation.
	
	The resolve averages (or otherwise combines) the sub-samples into the final color.

- **Benefits**
	
	**Smoother edges** on geometry.
	
	Better quality than no anti-aliasing, with less cost than full supersampling.
	
	Works automatically with rasterized primitives — no shader changes needed.

- **Limitations**
	
	Doesn’t reduce aliasing from **textures** or **shader calculations** (only geometry edges).
	
	Higher sample counts use more memory and bandwidth (since each pixel stores multiple samples).
	
	Requires a **multisampled framebuffer** (special VkImage with `samples = VK_SAMPLE_COUNT_4_BIT`, etc.).

- **Common Settings**
	
	`VK_SAMPLE_COUNT_1_BIT` → no multisampling.
	
	`VK_SAMPLE_COUNT_2_BIT`, `VK_SAMPLE_COUNT_4_BIT`, `VK_SAMPLE_COUNT_8_BIT` → common MSAA levels.
	
	Higher = smoother but more expensive.


**Analogy**:

Think of multisampling like **coloring inside the lines with several tiny brushes instead of one big brush**:  

- If part of a pixel is covered by a triangle, some of the tiny brushes paint it while others don’t.

- When you step back (resolve), the pixel looks partially filled — giving the appearance of a smoother edge.


In Vulkan, **Multisampling (MSAA)** reduces jagged edges by taking multiple coverage samples per pixel during rasterization. The fragment shader usually runs once per pixel, and the results are averaged during a **resolve step** to produce smoother final images.