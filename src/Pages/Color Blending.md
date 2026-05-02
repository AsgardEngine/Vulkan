**Color Blending** is a **fixed-function stage** that happens **after the Fragment Shader** has produced a color for each fragment, but **before writing to the framebuffer** (the image in memory that will become your final rendered scene).

Its purpose is to **combine the new fragment’s color output with the color already stored in the framebuffer**.

**Main Purposes of Color Blending**

**Transparency / Alpha Blending**:

- Allows drawing semi-transparent objects (glass, smoke, UI, etc.).

- Example: A fragment with 50% alpha blends with the background color.

**Compositing Effects**:

- You can configure how colors are combined mathematically (add, subtract, multiply, etc.).

- Useful for particle effects, lighting accumulation, and glow.

**Multiple Render Targets (MRTs)**

- If you write to multiple attachments, each one can have its own blending state.

### How It Works in Vulkan

Controlled via `VkPipelineColorBlendStateCreateInfo`.

For each framebuffer attachment, you can define:

- **Blend enable/disable**.
- **Blend factors** (e.g., source alpha, one minus source alpha).
- **Blend operations** (add, subtract, min, max).
- **Color write masks** (choose which channels R/G/B/A get updated).

**Classic alpha blending formula (when enabled):** 
	 
	  FinalColor = SrcColor * SrcFactor + DstColor * DstFactor
    
- `SrcColor` = output of Fragment Shader,
- `DstColor` = color already in framebuffer,
- `SrcFactor` & `DstFactor` are blending factors (like alpha, one, zero).


Example (common transparency):  

  SrcFactor = SrcAlpha
  DstFactor = 1 - SrcAlpha


**What Color Blending does *not***:

- It doesn’t compute colors — that’s the **Fragment Shader’s job**.
- It doesn’t decide whether a fragment is written — that’s handled by **depth/stencil tests**.
- It doesn’t apply post-processing — that’s usually another render pass.


**Analogy**:

Think of Color Blending like **painting with semi-transparent paint on a canvas**: 

- Each new stroke (fragment) can either cover, mix with, or highlight the existing paint (framebuffer contents).
- The blending settings are your "paint rules" (do you layer, mix, erase, etc.).

**In short:**  

In Vulkan, the **Color Blending stage** takes the output from the Fragment Shader and **combines it with the existing framebuffer color**, enabling transparency, compositing, and special effects.  
