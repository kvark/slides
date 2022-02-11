# wgpu-hal

Why another portable GPU abstraction?

Dzmitry Malyshau

---

Plan
1. **Positioning**
2. Architecture
3. State

---

## Goals
- portable subset of native APIs
- no overhead
- well aligned with WebGPU
- Rust all the way
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
3. State

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

### Data transfers

```rust
/// Copy from texture to buffer.
/// Works with a single array layer.
unsafe fn copy_texture_to_buffer<T>(
    &mut self,
    src: &A::Texture,
    src_usage: TextureUses,
    dst: &A::Buffer,
    regions: T,
) where
    T: Iterator<Item = BufferTextureCopy>;
```
Resolution: intersection of the APIs, but with flexibility of Vulkan.

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

### Other things

- shader modules
- queries
- presentable surfaces

---

Plan
1. Positioning
2. Architecture
3. **State**

---

## State

- [x] Vulkan
- [x] Metal
- [x] OpenGL ES-3
- [ ] D3D12

---

## Code

Each backend is roughly 5K LOC.

This is ~2x less than gfx-rs backends.

### Perf

Performance-wise, we can output 2x more sprites

for the same frame time, comparing to wgpu-core.
