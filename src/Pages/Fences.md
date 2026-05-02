A **fence** is a **GPU → CPU synchronization primitive**.

It is used when the **CPU needs to know when the GPU has finished some work**.

Unlike semaphores (which the GPU waits on), fences are **waited on by the host (CPU)**.

Think of a fence as a **“notification flag”** the GPU raises when it’s done.  

#  Purpose of Fences
  
**CPU waits for GPU completion**

- Example: Before reusing a command buffer, the CPU must know the GPU has finished executing it.
- Fence is signaled when the GPU completes that command buffer submission.

**Frame pacing / limiting**

- Prevents the CPU from running too far ahead of the GPU (triple buffering could cause runaway memory usage if unsynchronized).

**Resource management**

- Example: If the CPU uploads texture data, it should wait until the GPU is done using old data before overwriting it.

# How Fences Work

1- Create a fence (can start signaled or unsignaled).

2- Submit a command buffer with the fence.

3- When GPU finishes execution, it **signals** the fence.

4- CPU waits on the fence (`vkWaitForFences`) or checks it (`vkGetFenceStatus`).

5- Once done, the fence can be reset (`vkResetFences`) for reuse.

# Semaphores vs. Fences

| Feature            | **Semaphore** 🚦                   | **Fence** 🏁                     |
| ------------------ | ---------------------------------- | -------------------------------- |
| **Sync direction** | GPU → GPU                          | GPU → CPU                        |
| **Who waits?**     | GPU queues                         | CPU (host)                       |
| **Granularity**    | Work submission ordering           | Work completion notification     |
| **Common use**     | Render → Present, inter-queue sync | Frame pacing, CPU resource reuse |
| **Type**           | Binary / Timeline                  | Binary (signaled/unsignaled)     |
