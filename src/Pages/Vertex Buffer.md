
 A **vertex buffer** in Vulkan is simply a chunk of GPU memory that stores the **attributes of vertices** (points in 3D space that make up your geometry).

 Each vertex can have multiple attributes, such as:
 
- **Position** (x, y, z, w)
- **Normal vector** (for lighting)
- **Color**
- **Texture coordinates (UVs)**
- **Tangent, bitangent**, etc.


When you render, the GPU reads vertex data from the vertex buffer(s) and feeds it into the **vertex shader**.