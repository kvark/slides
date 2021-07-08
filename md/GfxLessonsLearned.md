---
marp: true
---

# gfx-rs: lessons learned

![logo](gfx-logo.png)

Dzmitry Malyshau
RustConf-2021

---

# Intro

What is gfx-rs?

<!-- ts: 0:30 -->
---

# Epoch: pre-ll

## Let's try to minimize a chance for errors!
- vertex layouts
- resource bindings
- render targets

<!-- ts: 1:00 -->

---

## Macros

```rust
gfx_defines!{
    vertex Vertex {
        pos: [f32; 4] = "a_Pos",
        tex_coord: [f32; 2] = "a_TexCoord",
    }
    pipeline pipe {
        vbuf: gfx::VertexBuffer<Vertex> = (), //<-- init-time portion
        out_color: gfx::RenderTarget<ColorFormat> = "Target0",
    }
}
...
let data = pipe::Data { // bind-time portion
    vbuf: vbuf, // it was long ago!
    out_color: window_targets.color,
};
```
<!-- ts: 2:00 -->
---

### Result
Users constantly misunderstanding the difference between macro-derived structures: one for initialization, one for run-time binding.

Macros were also very difficult to write and debug, not properly supported by tools.

### Lesson[1]
Stay away from macros, unless strictly necessary.

<!-- We used them because we were still learning the 
language (way before 1.0), no best practices were known. -->

<!-- ts: 3:00 -->
---

Function-style draw calls:
```rust
    /// Draws a `slice::Slice` using a pipeline state object, and its matching `Data` structure.
    pub fn draw<D: pso::PipelineData<R>>(&mut self, slice: &slice::Slice<R>,
                pipeline: &pso::PipelineState<R, D::Meta>, user_data: &D)
```
Most state is explicitly provided for each draw, then compared against the current to generate the state changes internally.

<!-- ts: 4:00 -->
---

### Result
Users saving the fat "data" objects and mutating them between the draws. Effectively, re-introducing the same state we tried to remove!

Their responsibility to fail, but still our problem in the API.
A good API shows the trail to success, not placing traps on the way.

Also, deducing the internal state changes means the overhead is not explicit, and is harder to reason about.

<!-- ts: 5:00 -->
---

### Lesson[2]
Functional style isn't universally better. It needs to be practical and match the target domain. Computer graphics is inherently stateful, and there is a performance cost to trying to change this.

Note: stack-based approach might have been better. [Luminance](https://github.com/phaazon/luminance-rs) has some of it.

<!-- ts: 5:30 -->
---

## Run-time knowledge

But what if we don't know this at compile time?..

Armed with type system, we can do it at run-time!
```rust
impl<'a> DataLink<'a> for RawRenderTarget {
    type Init = (&'a str, format::Format, state::ColorMask, Option<state::Blend>);
    ...
}
```
<!-- ts: 6:00 -->
---

![preoccupied scientists](preoccupied-scientists.jpg)

<!-- ts: 6:15 -->
---

### Result
Even more *confusion*!
Users basically followed the magic incantations without fully understanding them.

### Lesson[3]
Always build types *over* plain data, not the other way around.
If unsure, start with simple data and functions. Make typed layer optional.

<!-- In this case, the user should choose between using typed layer or not typed. Instead, they picked between the typed layer and the raw typed abomination over it. -->

<!-- ts: 7:15 -->
---

## Pipelines

In the API users defined graphics pipelines. This matches the concept in modern APIs, but at that point gfx pre-ll only had OpenGL support. Later it got DX11.

> We are ready for the next gen!

<!-- describe the idea and motivation for the pipelines -->

Is it safe? We hoped so!

<!-- ts: 8:30 -->
---

### Result
pre-ll pipelines never ended up being translated into the *real* graphics pipelines of modern APIs. There were other obstacles preventing the modern backends to be written (object lifetime on GPU, descriptors, etc).

### Lesson[4]
Don't design for the unknown future. Design for the known today instead.

There are many reasons for the design to need to change, and clarifying the future is just one of them.

<!-- very similar to an advice you can read on various
posts and threads, e.g. on HN -->

<!-- ts: 10:00 -->
--- 

# Epoch: gfx-hal

---

# Epoch: wgpu-hal

---

# lessons

- branding is important. Heritage needs to be preserved. Replacing the project on the same repository with a different one causes too much pain and confusion in the long run.

- types can be overwhelming and inefficient. They hinder debugging, compile times, API usability. Reaching safety with run-time checks is a perfectly valid path as well.

- being universal and unopinionated is *not* a feature. It's just an instrument to build something opinionated on top, which can actually be useful. But once it's build, there is an inevitable temptation to avoid compromises. Opinionated >> Universal.

- contributor/commit counts and project stats are unlikely important, at all.

- some contributions are harmful. Can't expect the authors to maintain their code. Smaller scope is a hidden advantage.

- clear vision helps, but it needs to reach to the end user. Building something low-level without a good understanding of higher levels is a recipe for disaster.

- low level primitives are as re-usable as narrow their API is. gfx-hal API is large.

- similarly, fighting against the flow is hard (re: gfx-portability).

- changing the project needs a new repo. Changing every bit of a ship confuses the users. Clear branding matters.

- don't put trust blindly into the open collaboration projects, like Khronoses APIs. The alternatives, developed behind closed doors but in tight collaboration with end users from both sides, can feature better designs. Examples: copy alignments, sub-passes, resource states.

---

# what's next?

gfx-rs repository is in maintenance mode.
But the ideas of gfx-rs live on within wgpu-hal and wgpu-rs.
