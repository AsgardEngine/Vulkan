Vulkan is an **API specification**. What happens in practice is this:


**Application (e.g., Game Engine / Game / Tool)**

- This is your code (Unreal, Unity, or a custom engine).
- It makes calls to the Vulkan API (like `vkCreateInstance`, `vkCmdDraw`, etc.).

**(Optional) Vulkan Layers**

- These are **interceptors** sitting between the app and the driver.
- They can validate API calls, log them, profile, or translate them.
- Example: `VK_LAYER_KHRONOS_validation`.

**Vulkan Loader (icd loader)**

- A system component (provided by LunarG or OS vendors).
- Dispatches Vulkan calls to the correct driver.
- Finds and activates layers requested by the app or developer.

**Vulkan Driver (ICD = Installable Client Driver)**

- Provided by the GPU vendor (NVIDIA, AMD, Intel, etc.).
- Implements Vulkan API calls on real hardware.
- Converts API calls into GPU instructions.

**GPU Hardware**

- Executes the commands on the actual graphics card.

Diagram (simplified)

  Game Engine / App
		↓
   [ Vulkan Layers ]   <-- optional (validation, debugging, profiling)
		↓
   Vulkan Loader
		↓
   Vulkan Driver (ICD)
		↓
	  GPU Hardware
    