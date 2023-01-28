---
title: Blade
description: Lean and mean graphics library.
slideOptions:
  theme: serif
  transition: 'fade'
---

# Blade

gfx, luminance, vulkano, wgpu, rafx, sierra ...
Yet *another* graphics library?

![](https://i.imgur.com/uXB8rPj.png)

---

## Motivation

Case: [Rusty Vangers](https://vange.rs/)

![](https://i.imgur.com/5G1SEDH.png)

  - 15 pipelines
  - 7 pipeline layouts, 8 bind group layouts
  - .. after [removing GPU physics](https://github.com/kvark/vange-rs/commit/c54426c3d6438dd1fd34e2d26147d06fff5065f9)

<!--
yes, but different
must be FUN (go through a process step by step)
other examples: Veloren
-->

---

### Motivation: GPU evolves

Lots workloads where CPU costs are small:
- AZDO (term from late OpenGL)
- compute/ML workloads
- Ray Tracing

<!--
Really, it's hard to come up with a counter-example.
Unless it's GTA-5.
-->

---

### Why not wgpu/hal?

Constraints...

<!--
wgpu: safety, layers, design constraints
wgpu-hal: attempted, but constrained by D3D12
-->

---

## Lean

![](https://i.imgur.com/DMX4T3z.png)

<!--
Who does the heavy lifting?
wgpu-hal, rafx-api, sierra: **user**
vulkano: user/types/macros/library
wgpu, rafx, arcana: **library**
blade: **driver**
-->

---

## State tracking

No per-object state. No image layout transitions.
Global barriers between passes.
```rust
if let mut pass_encoder = encoder.compute() {
    let mut pipeline_encoder = pass_encoder.with(pipeline);
    pipeline_encoder.dispatch(..);
}
```
<!-- all images are VK_LAYOUT_GENERAL -->

---

## State tracking: example

1. Create a resource
2. Use in a compute/render/transder pass
3. Submit the command encoder.

---

## Binding model

No layouts. No descriptor sets.
No uniform buffers, just plain data.

<!-- no headache, no caching -->

---

### Binding model: define group

```rust
#[derive(bytemuck::Pod]
struct MyUniformStruct {
    data: u32
}
#[derive(blade::ShaderData)]
struct Foo {
    texture: blade::TextureView,
    uniform: MyUniformStruct,
}
```

---

### Binding model: pipeline creation

```rust
let foo_layout = <Foo as blade::ShaderData>::layout();
context.create_pipeline(blade::ComputePipelineDescriptor {
  data_layouts: &[&foo_layout],
});
```

---

### Binding model: binding

```rust
pass.bind(0, &Foo {
    texture: my_texture_view,
    uniform: MyUniformStruct { // plain data
        data: 1,
    },
});
```

---

### Binding model: thoughts

Makes prototyping a breeze!

Was BG/BGL/PL a design mistake in WebGPU?

<!-- without it, WebGPU basically becomes Metal -->

---

### Lean: code size

![](https://i.imgur.com/pwjFXUX.png)

Disclaimer: Blade is much incomplete!

---

## But is it fast?

Can do around 20K draw calls per frame on a low-end laptop.

---

## Mean

- Totally *unsafe*: use at your own risk.
- No tools: ask your local GPU vendor.
- Incomplete: customize for your needs.

<!-- use your own validation and capturing -->

---

### Mean: shortcuts

Who needs vertex buffers, anyway?

One context - one window.

---

### Mean: no contract

All libraries: we got you!
Blade: bring your own maintainer

See [What it feels like to be an open-source maintainer](https://nolanlawson.com/2017/03/05/what-it-feels-like-to-be-an-open-source-maintainer/)

---

## Ergonomics

Resource objects: light and copyable.

```rust
pub struct Buffer {
    raw: vk::Buffer,
    memory_handle: usize,
    mapped_data: *mut u8,
}
```

Context: Has everything, internally synchronized.

<!--For an unsafe library, ergonomics >> partial safety.-->

---

## API: Shaders

Validated(*) WGSL modules.
Without resource binding decorations!
```rust
var sprite_texture: texture_2d<f32>;
var sprite_sampler: sampler;
```

---

## TL;DR: why use Blade?

![](https://i.imgur.com/Hp9gqlV.png)


---

## Why not use Blade?

![](https://i.imgur.com/ap9l0vj.png)

---

# Demo & Questions

https://github.com/kvark/blade
