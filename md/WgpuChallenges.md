---
marp: true
theme: uncover
---
<!-- footer: '' -->

# wGPU Challenges

Dzmitry Malyshau, 2021

---
<!-- footer: '' -->

## About Me

![Firefox/WebRender](WgpuChallenges/FirefoxDebug.jpg) ![GTA](WgpuChallenges/GTA5.jpg) ![Treasure Hunt](WgpuChallenges/JVL-TreasureHunt.jpg)

- _Mozilla Firefox_
- _Rockstar Games_
- _JVL Inc game machines_
- bunch of other things...

---
<!-- footer: '' -->

![WebGPU Problems](WgpuChallenges/webgpu-problem.png)

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
<!-- footer: '' -->

![Platforms map](ProcessingShaders/PlatformsMap.jpeg)

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

![Vulkanized pain points](WgpuChallenges/vulkanized-pain-points.png)

<!-- footer: '
  ... a lot of code
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

Effort: 2 months to build from scratch.

---
<!-- footer: '' -->

## Problem-2: Shading language

No text SPIR-V, no C-like syntax.

Hard safety and portability requirements.

```rust
[[stage(fragment)]]
fn fs_extra() -> [[location(0)]] vec4<f32> {
    return vec4<f32>(0.0, 0.5, 0.0, 0.5);
}
```

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

![Naga architecture](WgpuChallenges/naga-architecture.png)

---
<!-- footer: '' -->

### Native Path: Results

![WebGPU IR path results](WgpuChallenges/wgpu-ir-path.png)

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
<!-- footer: '' -->

### Solution-3b: Command Injection

![wgpu sync points](WgpuChallenges/wgpu-sync.png)

<!-- footer: '
  Creating many small command buffers.
' -->
---
<!-- footer: '' -->

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

### Results

- the only safe GPU abstraction in town
- runs on everything (*)
- accessible from Native (*) and Web

<!-- footer: '
 ... with help of Angle currently

 Has bindings for C, D, Python, Java, Julia, etc
' -->
---
<!-- footer: '' -->

## Lesson-1

> Abstraction has to be driven by something.

![Vangers raymip](WgpuChallenges/vangers-raymax-debug.png)

<!-- footer: '
Anticipating future use isn't helpful.
' -->
---
<!-- footer: '' -->

## Lesson-2

> Re-inventing things has its place.

![wgpu Encounter](WgpuChallenges/wgpu-encounter.jpg)

<!-- footer: '
Existing solutions, like libraries, or standards,
aren't always a good fit.
' -->
---
<!-- footer: '' -->

## Lesson-3

> Community power is misleading.

![wgpu Spaceship](WgpuChallenges/wgpu-spaceship.png)

<!-- footer: '
The only unstoppable power is a motivated individual.
' -->
---
<!-- footer: '' -->

![fancy picture](WgpuChallenges/grass-field.png)

# Questions?
