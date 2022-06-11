---
title: Choir
description: Task Orchestration Framework.
---

# Choir

Task Orchestration Framework

> Orchestration: the planning or coordination of the elements of a situation to produce a desired effect, especially surreptitiously

---

![destiny task graph](https://i.imgur.com/iXU9Dpq.jpg)

<!-- inspiration example, 2015 -->

---

## Why?

- give: control flow (at macro level)
- get: free scaling (both perf and complexity)

@Ralith:
> any time you take control flow away from the user you have a really high usefulness bar to clear

---

![tasks everywhere meme](https://i.imgflip.com/6iycp0.jpg)

<!-- not about the technology -->

---

## Scalable?

- Way to organize your thoughts, and code
- Separating scheduling from structure

---

## Example
```rust
let mut choir = choir::Choir::new();
let _worker1 = choir.add_worker("worker1");
let first = choir.add_task(|| { println!("First"); }).run();
let mut second = choir.add_task(|| { println!("Second"); });
second.depend_on(&first);
```

---

![scene loading graph](https://i.imgur.com/bMiiqXu.png)

---

## What is the problem?

<!-- ok, we understand the idea,
but what problem is it really solving?
-->

Rust + ECS = ...

<!-- very nice combination, but overused -->

> if you have an advanced ECS, everything looks like a system

---

## ECS-centric approach

![](https://i.imgur.com/k8LlilB.png)

<!-- all logic is systems, all data is components and resources.
"Everything is a system" is a very narrow view, introduces friction.
-->

---

## Task-centric approach

![](https://i.imgur.com/MIyOuvn.png)

<!-- the beauty is - task orchestrator not only schedules ECS jobs, but also any other jobs, seamlessly
-->
<!-- it's a lower level approach to scheduling -->

---

## Choir features

<!-- not technically complex -->

- task dependencies, including from running tasks
- spawning tasks from tasks
- multi- and iter- tasks
- flexible workers

---

![Arc dependencies](https://i.imgur.com/Yjqhl9H.png)

---

## Perf

Executing 100k(!) empty tasks:
- individually: 28ms
- as a multi-task: 6ms

Each task: 60ns to 280ns

---

## Future

- circular dependency detection :dizzy: 
- exhaustive testing with Loom :radioactive_sign: 
- scheduler for Hecs :gear: 

---

![map of control flow](https://i.imgur.com/w1acEhC.png)


---

## Happy tasking!

https://github.com/kvark/choir
