---
marp: true
theme: uncover
---
<!-- footer: '' -->

# wGPU Challenges

Dzmitry Malyshau

Graphics Engineer @Mozilla,
former Rockstar

---
<!-- footer: '' -->

## WebGPU: Requirements

  - safe and secure
  - works on all modern platforms
  - behaves portably
  - (bonus) comes with a shading language!

---
<!-- footer: '' -->

## Problem-1: Cross APIs

![gecko task](WgpuChallenges/wgpu-pre-graph.png)

---
<!-- -->

Available resources:
  - 1 engineer + community
  - https://github.com/gfx-rs/gfx
  - VkPortability wrapping

![Vulkan hammer](WgpuChallenges/khronos-2017-vulkan-alt-cropped.png)

<!-- footer: '
  When all you have is Vulkan, everything looks like a Portability problem.
' -->
---
<!-- footer: '' -->

### Solution-1-I: Vulkanized

![Vulkan-centric architecture](WgpuChallenges/wgpu-old-graph.png)

---
<!-- footer: '' -->

Effort: 6-12 months to build into the browser.
Using gfx-portability and iterating on it.

### Good:
  - runs well on Vulkan
  - can use existing tooling, like SPIRV-Cross

---
<!-- footer: '' -->

#### Vulkanized: Pain Points
  - Vulkan -> anything isn't that great
  - WebGPU -> Vulkan loses some important bits
  - still need a new WGSL frontend
  - hard to integrate with SPIRV-Cross efficiently
  - dependencies sppread across repos
  - a lot of code!

<!-- footer: '
Example: memory placement. Vulkan -> anything has to pretend it works,
but WebGPU -> Vulkan could just use heaps in a restricted way.

Similar example - command pools.
Another example - command buffer reusability.
Important bits lost: compute passes.
' -->

---
<!-- footer: '' -->

### Solution-1-II: Native Implementation

![WebGPU-centric architecture](WgpuChallenges/wgpu-new-graph.png)

---
<!-- footer: '' -->

[Hardware Abstraction Layer](https://github.com/gfx-rs/wgpu/tree/master/wgpu-hal)

- explicit barriers
- explicit command allocators
- fully unsafe
- near-zero overhead
- efficient API intersection
- shares types with the rest of `wgpu`

---
<!-- footer: '' -->

![wgpu logo](WgpuChallenges/wgpu-logo.png)

Effort: ~2 months to build from scratch.

---
<!-- footer: '' -->

## Problem-2: Shading language

No text SPIR-V, no C-like syntax.

Hard safety and portability requirements.

---
<!-- footer: '' -->

### Solution-2: Naga

In-house [shader translation](https://github.com/gfx-rs/naga) from anything to anything.
Lightning fast! Roughly 4x faster than SPIRV-Cross.

First-class support for WGSL.

Proof of concept took 1 month. Development is ongoing.

<!-- footer: '
  Turns out, writing from scratch isnt that difficult!
  Or so it seemed at start.
' -->
---
<!-- footer: '' -->

Result: 2x reduction in code size.
Less pain to maintain, develop, and debug.
More efficient on DX12 and Metal.

---
<!-- footer: '' -->

## Problem-3: Implicit synchronization

- like D3D11, Metal, or OpenGL, but specified
- or like D3D12, but with full *implicit promotion/decay*

---
<!-- footer: '' -->

### Solution-3a: Usage Groups

![wgpu texture usage](WgpuChallenges/wgpu-usages.png)

---
<!-- -->

### Solution-3b: Command Injection

![wgpu sync points](WgpuChallenges/wgpu-sync.png)

<!-- footer: '
  Creating many small command buffers.
' -->
---
<!-- -->

## Problem-4: Multi-threading

- recording command buffers on different threads
- creating resources on different threads
- easy to get into a deadlock

---
<!-- footer: '' -->

#### Solution-4: Access Order

![using the same mip from different encoders](WgpuChallenges/wgpu-sync-order.png)

---
<!-- footer: '' -->

## Lesson-1

> Abstraction has to be driven by something.

Anticipating future use isn't helpful.

---
<!-- footer: '' -->

## Lesson-2

> Re-inventing things has its place.

Existing solutions, like libraries, or standards,
aren't always a good fit.

---
<!-- footer: '' -->

## Lesson-3

> Community power is misleading.

The only unstoppable power is a motivated individual.

---
<!-- footer: '' -->

![fancy picture](WgpuChallenges/grass-field.png)

# Questions?
