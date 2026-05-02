
 An **index buffer** is another GPU buffer, but instead of storing vertex attributes, it stores **indices (integer references)** pointing into the vertex buffer.
 
This allows you to **reuse vertices** efficiently.

For example, a cube has 8 unique corners (vertices), but if drawn with triangles, it takes 36 vertices (12 triangles × 3 vertices). With an index buffer, you can store just the 8 vertices once, and use indices to assemble all 12 triangles.  

 **Purpose**:

- Saves memory (no duplicate vertex data).
- Improves cache locality and performance on the GPU.
- Makes geometry representation cleaner.

**Without vs With Index Buffer**:

- **Without index buffer**: GPU reads vertex buffer linearly (every group of 3 vertices = a triangle).

- **With index buffer**: GPU looks up vertex attributes using the indices (e.g., `0, 1, 2, 2, 3, 0` defines two triangles sharing an edge).
