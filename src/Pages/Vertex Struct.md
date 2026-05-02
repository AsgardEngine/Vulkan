# The Struct

```
  struct Vertex {
	glm::vec2 pos;   // 2D position (x, y)
	glm::vec3 color; // RGB color (r, g, b)
	...
  };
```
    
**`glm::vec2 pos`**: Holds 2 floats → the vertex position in 2D space.
**`glm::vec3 color`**: Holds 3 floats → the vertex color (red, green, blue).

So each vertex is **5 floats = 20 bytes total**.  

# Binding Description
   
```
static vk::VertexInputBindingDescription getBindingDescription() {
	return { 
		0,                     // binding index (used in shaders)
		sizeof(Vertex),        // stride: size of one vertex in bytes
		vk::VertexInputRate::eVertex // input rate: one per vertex (not per instance)
	};
}
```

**binding = 0**: This binding index associates vertex buffer data with the shader’s input layout.

**stride = sizeof(Vertex)**: Distance (in bytes) between consecutive vertices in memory (here 20 bytes).

**`eVertex`**: Means the buffer advances **per vertex**. (If you set `eInstance`, it would advance per instance, used in instanced rendering).

**Purpose**: Describes _how to step through vertex data in the buffer_.  

# Attribute Descriptions
    
      
```    
static std::array getAttributeDescriptions() {
	return {
		// Attribute 0: position
		vk::VertexInputAttributeDescription(
			0,  // location in shader
			0,  // binding index (matches getBindingDescription binding = 0)
			vk::Format::eR32G32Sfloat, // vec2 -> 2 floats
			offsetof(Vertex, pos)      // memory offset of pos inside struct
		),
		// Attribute 1: color
		vk::VertexInputAttributeDescription(
			1,  // location in shader
			0,  // same binding
			vk::Format::eR32G32B32Sfloat, // vec3 -> 3 floats
			offsetof(Vertex, color)       // offset of color inside struct
		)
	};
}
   
```

Each attribute links **shader input variables** (like `layout(location = 0) in vec2 inPos;`) to memory inside the vertex.

- **location**: Matches the shader’s `layout(location = N)` qualifier.
- **binding**: Which vertex buffer binding this attribute comes from (here, binding = 0).
- **format**: Defines how many components and their type (vec2 = 2 floats, vec3 = 3 floats).
- **offset**: Byte offset in the struct where the attribute starts.
  
So:  

- Attribute 0 → position (`pos`), stored as two floats.
- Attribute 1 → color (`color`), stored as three floats.


# 4. Shader Correspondence

Given this struct, your vertex shader would expect:  


  layout(location = 0) in vec2 inPos;
  layout(location = 1) in vec3 inColor;
  
---

- `Vertex` defines CPU-side data layout.
- `getBindingDescription()` tells Vulkan how to step through the buffer.
- `getAttributeDescriptions()` maps struct members → shader inputs by location.
