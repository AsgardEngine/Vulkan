
In Vulkan (and in graphics pipelines generally), **Rasterization** is the **fixed-function stage** that takes **primitives (points, lines, triangles)** output by the Vertex/Tessellation/Geometry Shader stages and converts them into **fragments (potential pixels)** that the **fragment shader** can process.  
  
It’s basically the process of turning vector geometry into a pixel representation on the framebuffer.  
## Main Purposes of Rasterization

- **Primitive Clipping**
	
	Cuts away portions of primitives that lie outside the viewport or user-defined clip regions.
	Ensures only visible geometry is processed further.

- **Primitive Culling**
	
	Can discard back-facing triangles (backface culling) to save work.
	Controlled via `VkPipelineRasterizationStateCreateInfo`.

- **Viewport & Scissor Transformation**
	
	Maps vertex positions (from normalized device coordinates) into **window coordinates** (framebuffer pixels).
	Applies scissor rectangles to restrict rendering to a subregion of the framebuffer.

- **Fragment Generation**
	
	Breaks triangles/lines/points into **fragments** (candidate pixels).
	Each fragment carries interpolated attributes (color, texture coords, normals, etc.) from the vertices.

- **Depth & Face Rules**
	
	Assigns depth values to fragments.
	Handles rules like polygon mode (fill, line, point).


**What Rasterization does *not***

- It does **not shade fragments** → that’s the job of the **Fragment Shader**.
- It does **not perform blending, depth testing, or writing to the framebuffer** → those happen in the **Output Merger** (later stage).


**Analogy**:

Think of rasterization as **projecting a wireframe model onto graph paper**:  

- The model (triangles) is drawn on a grid (pixels).
- The lines/triangles are “colored in” by generating little squares (fragments).
- Each fragment is then handed off for shading (lighting, textures, etc.).

In Vulkan, **Rasterization converts primitives into screen-space fragments**. It’s the bridge between geometric processing (vertex/tessellation/geometry shaders) and pixel processing (fragment shaders).  
