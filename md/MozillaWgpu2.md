---
marp: true
---

# wgpu project

![logo](wgpu-logo.png)

WebGPU implementation in Rust

Dzmitry Malyshau, 2021

---

# Project goals
- the *fastest*, most *robust*, and *portable* implementation of WebGPU
  - Vulkan, D3D12/D3D11, Metal, and OpenGL ES3
- integrates into browsers, can also be run standalone

--- 

# Mozilla-aligned goals
- power WebGPU in Gecko (and previously Servo)
- serve as a prototyping ground to test and generate new ideas for WebGPU and WGSL specifications
  - the only alternative implementation to Googls's Dawn on many platforms!
- battle-test the new graphics stack to potentially move WebRender on, in the future

---

# General info
- Age: ~3 years old
- Contributors: Dzmitry + ~100 from the community
- Commits: 1-2 commits daily on average
- Language: 100% Rust

---

# Community adoption
- [Deno](https://deno.land/) - "A secure runtime for JavaScript and TypeScript"
- [Bevy](https://bevyengine.org/) - "A refreshingly simple data-driven game engine built in Rust"
- [lots of others](https://github.com/gfx-rs/wgpu-rs/wiki/Applications-and-Libraries)

---

# Tech stack

![big picture](https://gfx-rs.github.io/img/wgpu-big-picture.svg)

---

# Code size
| component | LOC | needed by Gecko? |
| --------- | --- | ---------------- |
| core      | 27k | yes              |
| rust API  | 15k | no, but coupled  |
| gfx-hal   | 68k | mostly           |
| naga      | 37k | mostly           |

---

# Mozilla-central upstream

Pros:
- quick to address issues (limited to core only)
- tracking through Bugzilla, landing via Phabricator (also core only)
- testing on WebGPU CTS

Cons:
- killing external contribution
- effectively *forking* `wgpu`
- increasing merge latency from 15 minutes to up to 24 hours

---

# GitHub upstream
- can also run WebGPU CTS on CI - minimizes chance of Gecko breakage
- well-defined API boundary (unlike WebRender) - reduces integration breakage
- keeps the community momentum, lowe entry barrier, and benefits from external work
- there is still a workflow for urgent fixes: via a fork an a cargo override in Gecko. Works for any component, not just the core.
- we already do this in Neqo, Rkv, and other projects
