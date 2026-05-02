
In **Vulkan**, a **Uniform Buffer** (often called a _Uniform Buffer Object_ or **UBO**) is a special type of GPU buffer that stores **read-only data shared across many shader invocations**.

# Key Purposes

- **Provide constant data to shaders**

- Uniform buffers hold small, frequently accessed data such as:
	
	- Transformation matrices (model, view, projection)
	- Lighting parameters
	- Material properties
	- Global settings (time, resolution, etc.)

- This data is the same (or mostly the same) for many vertices/fragments, unlike vertex attributes which change per vertex.

- **Efficient sharing of data**

- Instead of pushing constants individually, you place them in a uniform buffer.

- Multiple shaders and pipeline stages can read from the same buffer without duplicating data.

- **Better performance vs. push constants (for larger data)**

- Vulkan’s **push constants** are extremely fast but limited in size (usually 128 bytes).

- For larger but still read-mostly data, uniform buffers are the right tool.

# **Dynamic updates**

- Uniform buffers can be updated per frame or per draw call.

- With **dynamic uniform buffers**, you can store multiple sets of uniforms in one buffer and index into them per draw—reducing the need for many descriptor sets.


## Example
  
A uniform buffer might hold:  

// Example uniform buffer data
struct UBO {
	mat4 model;
	mat4 view;
	mat4 proj;
	vec4 lightPosition;
};

Every shader invocation (vertex or fragment) can then access `ubo.model`, `ubo.view`, etc., without each vertex or pixel needing its own copy.  

## Summary

Uniform buffers in Vulkan are a mechanism for:  

- Storing **small, read-only, shared data** accessible by shaders.
- Avoiding redundant per-vertex/per-fragment storage of the same constants.
- Handling larger sets of uniform data than push constants allow.
- Efficiently managing per-frame or per-draw call state.
