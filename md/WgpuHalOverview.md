---
marp: true
---

# wgpu-hal

Why another portable GPU abstraction?

Dzmitry Malyshau

---

Plan
1. **Positioning**
2. Architecture
3. Workflow

---

## Goals
- portable subset of native APIs
- no overhead
- well aligned with WebGPU
- fully unsafe, but still usable

---

### Differences with gfx-hal:
- much reduced scope, does not try to include everything
- designed to power wgpu-core (the high level API for it), not in the abstract
- sharing types (wgpu-types) with wgpu-core
- not over-engineered (less engineered?)

---

Example from gfx-hal at some point:
```rust
    unsafe fn flush_mapped_memory_ranges<'a, I>(&self, ranges: I) -> Result<(), OutOfMemory>
    where
        I: IntoIterator,
        I::Item: Borrow<(&'a B::Memory, Segment)>;
        I: IntoIterator<Item = (&'a B::Memory, Segment)>,
        I::IntoIter: ExactSizeIterator;
```

---

Same with wgpu-hal today:
```rust
unsafe fn invalidate_mapped_ranges<I>(&self, buffer: &A::Buffer, ranges: I)
    where
        I: Iterator<Item = MemoryRange>;
```

---

Plan
1. Positioning
2. **Architecture**
3. Workflow

---

Traits as API contracts:
```rust
pub trait Api: Clone + Sized {
    type Instance: Instance<Self>;
    type Surface: Surface<Self>;
    type Adapter: Adapter<Self>;
    type Device: Device<Self>;

    type Queue: Queue<Self>;
    type CommandEncoder: CommandEncoder<Self>;
    type CommandBuffer: Send + Sync;

    type Buffer: fmt::Debug + Send + Sync + 'static;
    type Texture: fmt::Debug + Send + Sync + 'static;
    ...
```
Same as gfx-hal, but different from Rafx.

---

### Binding model:
```rust
    /// Sets the bind group at `index` to `group`, assuming the layout
    /// of all the preceeding groups to be taken from `layout`.
    unsafe fn set_bind_group(
        &mut self,
        layout: &A::PipelineLayout,
        index: u32,
        group: &A::BindGroup,
        dynamic_offsets: &[wgt::DynamicOffset],
    );
```
Resolution: Vulkan, but with WebGPU caveats

---

### Passes
```rust
unsafe fn begin_render_pass(&mut self, desc: &RenderPassDescriptor<A>);
unsafe fn end_render_pass(&mut self);
unsafe fn begin_compute_pass(&mut self, desc: &ComputePassDescriptor);
unsafe fn end_compute_pass(&mut self);
```
Resolution: WebGPU, closest to Metal

---

### Transition barriers:
```rust
pub struct TextureUse: u32 {
    const COPY_SRC = 1;
    const COPY_DST = 2;
    const SAMPLED = 4;
    const COLOR_TARGET = 8;
    const DEPTH_STENCIL_READ = 16;
    const DEPTH_STENCIL_WRITE = 32;
    const STORAGE_LOAD = 64;
    const STORAGE_STORE = 128;
    /// The combination of all read-only usages.
    const READ_ALL = ...;
    /// The combination of all write-only and read-write usages.
    const WRITE_ALL = ...;
}
```
Resulution: closest to D3D12 model, but with WebGPU states.

---

### Command pools:
```rust
pub trait CommandEncoder<A: Api>: Send + Sync {
    /// Begin encoding a new command buffer.
    unsafe fn begin_encoding(&mut self, label: Label) -> Result<(), DeviceError>;
    /// Discard currently recorded list, if any.
    unsafe fn discard_encoding(&mut self);
    unsafe fn end_encoding(&mut self) -> Result<A::CommandBuffer, DeviceError>;
    /// Reclaims all resources that are allocated for this encoder.
    /// Must get all of the produced command buffers back,
    /// and they must not be used by GPU at this moment.
    unsafe fn reset_all<I>(&mut self, command_buffers: I)
    ...
    // actual commands to encode
}
```
Resulution: D3D12 model with much nicer types.

---

Plan
1. Positioning
2. Architecture
3. **Workflow**

---

## Module structure

`mod.rs`:
```rust
impl crate::Api for Api {...}

pub struct Device { // public if they have to be
    shared: Arc<AdapterShared>, // all shared
    features: wgt::Features,
}
```
`device.rs`:
```rust
impl crate::Device for super::Device {...}
```

---

## Clippy
```rust
#![allow(
    clippy::match_like_matches_macro, // We don't use syntax sugar where it's not necessary.
    clippy::redundant_pattern_matching, // Redundant matching is more explicit.
    clippy::single_match, // Matches are good and extendable, no need to make an exception here.
    clippy::needless_lifetimes, // Explicit lifetimes are often easier to reason about.
    clippy::new_without_default, // No need for defaults in the internal types.
    clippy::vec_init_then_push, // Push commands are more regular than macros.
)]
#![warn(
    trivial_casts,
    trivial_numeric_casts,
    unused_extern_crates,
    unused_qualifications,
    // We don't match on a reference, unless required.
    clippy::pattern_type_mismatch,
)]
```

---

## Benchmarks

roughly 2x more sprites for the same frame time, comparing to wgpu-core