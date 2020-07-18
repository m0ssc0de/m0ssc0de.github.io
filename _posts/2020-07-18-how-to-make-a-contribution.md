---
layout: post
title: How to make a contribution to Embark Studios' opensource community
date: 2020-07-18 20:55
category:
author: m0ssc0de
tags: [embark, rust, opensource]
summary:
---

The name of "Embark Studios" was unfamiliar for me at first. But by some searches on Google, I found they are "old friends".
So I write this blog and want to make a contribution to them.

The team of Embark Studios came from EA and DICE. They made "Battlefield" which I spend lots of weekends at high school.
I can't forget the feelings of reversing the situation in "Battlefield". Usually, in the beginning, we stay at the last control point.
enemy's helicopter was hovering in the air. Lots of vehicles rush in. With the cooperation, we go through the defense hardly.
Then get another one control point. Then two, then three. Oh, that pressure and concentration consume so many hormones.
I miss those guys who fought together. And the investor of Embark is familiar too. "NEXON“ made "Kart rider" which I played a lot too.
My brain has been full of its iconic background music and brake sound effects right now. I miss the time when we hang out to internet cafes.

Put away thoughts. Because of my career habits, I search for The Embark Studios on Github. There are many Rust projects in their account.
With reading their introduction, I know they choose rust as the primary language at the beginning. I did the same on my side projects.
Holy, they are amazing. I can not wait for joining them if I qualify.

I know Embark Studios is world-class. There is a high probability that I am not qualified.
But there is always something I can do and something worth learning. I don't work in the game industry.
So maybe it's easy to make a contribution to their opensource community for learning the basics in the video game industry.
And then I found I used their opensource project, "wg-ui". My best thanks to Embark Studios.
I should have learned the author of the project which I used next time. Let's have a look at the Embark Studios' projects.
And find out how I can make a contribution to them.

The [rust-ecosystem](https://github.com/EmbarkStudios/rust-ecosystem) list projects written by Rust.

| Name                                                                            | Description                                                                                             |
|---------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| 🌋 [`ash-molten`](https://github.com/EmbarkStudios/ash-molten.git)              | Statically linked MoltenVK for Vulkan on Mac using Ash                                                  |
| 👷 [`buildkite-jobify`](https://github.com/EmbarkStudios/buildkite-jobify)      | Kubekite, but in Rust, using configuration from your repos                                              |
| 📜 [`cargo-about`](https://github.com/EmbarkStudios/cargo-about)                | Cargo plugin to generate list of all licenses for a crate                                               |
| ❌ [`cargo-deny`](https://github.com/EmbarkStudios/cargo-deny)                   | Cargo plugin to help you manage large dependency graphs                                                 |
| 🎁 [`cargo-fetcher`](https://github.com/EmbarkStudios/cargo-fetcher)            | `cargo fetch` alternative for use in CI or other "clean" environments                                   |
| 📦 [`krates`](https://github.com/EmbarkStudios/krates)                          | Creates graphs of crates from cargo metadata                                                            |
| 🎳 [`physx`](https://github.com/EmbarkStudios/physx-rs)                         | Use [NVIDIA PhysX](https://github.com/NVIDIAGameWorks/PhysX) in Rust                                    |
| 🐏 [`rpmalloc-rs`](https://github.com/EmbarkStudios/rpmalloc-rs)                | Cross-platform Rust global memory allocator using [rpmalloc](https://github.com/rampantpixels/rpmalloc) |
| 🆔 [`spdx`](https://github.com/EmbarkStudios/spdx)                              | Helper crate for SPDX expressions                                                                       |
| 🔆 [`superluminal-perf`](https://github.com/EmbarkStudios/superluminal-perf-rs) | [Superluminal Performance](http://superluminal.eu) profiler integration                                 |
| 📂 [`tame-gcs`](https://github.com/EmbarkStudios/tame-gcs)                      | Google Cloud Storage functions that follows the sans-io approach                                        |
| 🔐 [`tame-oauth`](https://github.com/EmbarkStudios/tame-oauth)                  | Small OAuth crate that follows the sans-io approach                                                     |
| 🎨 [`texture-synthesis`](https://github.com/EmbarkStudios/texture-synthesis)    | Example-based texture synthesis generator and CLI example                                               |

Some of the words are new to me. By doing research, I paraphrase them below.

Vulkan is a low-level graphics API. Its function is similar to OpenGL but performs better. Use the same API on different platforms and os. More small overhead on CPU and GPU driver. Better support multi-cores. Pre-compiled interface.

MoltenVK is a layer to implement the Vulkan interface for working on Apple's Metal. Metal is a graphics API on Apple's devices, macOS, iOS, tvOS. MoltenVK trans SPIR-V(Vulkan) to MSL(Metal).

Ash is a rust wrapper (or lib?) of Vulkan. `ash-molten` is used to static link to MoltenVK.

Buildkite is a CI SaaS, similar to Drone.io. But can not be on-premise.

`kubekite` could fetch Buildkite job and run it as a k8s job.

`buildkite-jobify` could be considered as a Rust version of `kubekite`.

`cargo-about` is used to manage project licenses and dependencies' licenses. One of its dependencies, rpmalloc-sys, looks like need gcc >=4.9.

`cargo-deny` could help us check dependencies in several ways.

`cargo-fetch` could use third party storage to cache crate.io.

`krates` could graph the relationship of all dependencies by `Cargo.toml`
It looks like need to update semver's version. There are two related PR didn't execute.
I don't know why. Maybe there are other reasons.

`NVIDIA PhysX` is a realtime physics engine middleware.
`physx-rs` is the rust wrapper of NVIDIA PhysX SDK.

`rpmalloc-rs` is a cross-platform Rust global memory allocator.

`SPDX` response to licenses and copyrights of software.

`superluminal` is a profiling tool.

`sans I/O` is a kind of network protocol to support both synchronous and asynchronous.

`tame-gcs`, `tame-oauth` are both a `sans I/O` approach in their own field.

`texture-synthesis` is to generate pictures for texture map.

`wg-ui` is the most familiar one. It's used to managed WatchGuard config easily in the web page.

Then we can simply classify them.

Some are about graphs and physic.
- ash-molten
- physx
- texture-synthesis

Rust base tools
- cargo-about
- cargo-deny
- cargo-fetcher
- krates

Perfoamce
- rpmalloc-rs
- superluminal-perf
- tame-gcs
- tame-oauth

Automation
- buildkite-jobify
- wg-ui

Others
- spdx

The automation part is easier for me. So I want to start from the `wg-ui` like the list below.

- wg-ui
- buildkit-jobify
- cargo-fetcher

Took a quick look at their issues zone. There are some issues I really can make help to.

In the next blog, I will try to analyze source code and find some requirements to complete.
If it fails to help in the end. A lot of things will be learned throughout the process.