---
marp: true
---

# gfx-rs: lessons learned

![logo](gfx-logo.png)

Dzmitry Malyshau, 2021

---

# pre-ll epoch

--- 

# hal epoch

---

# wgpu-hal

---

# lessons

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