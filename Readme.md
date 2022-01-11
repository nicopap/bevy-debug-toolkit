# Bevy debug toolkit

[![Bevy tracking](https://img.shields.io/badge/Bevy%20tracking-released%20version-lightblue)](https://github.com/bevyengine/bevy/blob/main/docs/plugins_guidelines.md#main-branch-tracking)
[![Latest version](https://img.shields.io/crates/v/bevy_debug_toolkit.svg)](https://crates.io/crates/bevy_debug_toolkit)
[![MIT/Apache 2.0](https://img.shields.io/badge/license-MIT%2FApache-blue.svg)](./LICENSE)
[![Documentation](https://docs.rs/bevy-debug-toolkit/badge.svg)](https://docs.rs/bevy-debug-toolkit/)

An unified collection of crates for visual debugging.

Features:
- [ ] Overlay `println!`
- [ ] debug lines based on [bevy_debug_lines](https://github.com/Toqozz/bevy_debug_lines)
- [ ] debug shapes based on [bevy gizmo](https://github.com/lassade/bevy_gizmos/)
- [ ] Billboard text (blocked until [render to texture is implemented](https://github.com/bevyengine/bevy/pull/3412))

## Philosophy

### Debug-oriented API

The crates define a set of macros you can call inline to draw stuff, so that
you don't have to modify the argument lists of your systems or propagate down
the call stack `ResMut`s (and conversely remove them when you decide you don't
need the debug view anymore). We also use concurrent data structures so that
it's possible to call the draw macros in concurrent threads without congestion.
This is better than a `ResMut` since multiple systems depending on the same
`ResMut` cannot run concurrently, it would force the scheduler to serialize all
systems that have access to the same `ResMut`.

The solution adopted is to use an append-only command buffer, one for each call
site + thread ID (because the same code could run concurrently on different
threads) a `DashMap<(CallSite, ThreadId), DebugDrawCommand>` does the trick.
(Practical concern: may be way too sparse and kill perf through cache misses,
needs profiling)

The crate also has a feature to turn macro calls into no-ops. 

### Debug layers

You can tag all your debug objects with a layer. Layers are just collections of
objects to display in a consistent way. You can enable/disable layers through
a `ResMut<DebugLayers>`, multiple layers can be active at the same time.

the crate provides an API and default input systems to control the layers. But
you can skip it and implement your own.

### Unified API

When relevant, the API is consistent, we try to generally provide the following
features:

* Individually enable/disable depth culling for each debug objects
* Coloring
* Retained/Immediate style "timeout"

## License

The code is licensed under either MIT or Apache 2.0, at your leisure. See LICENSE
file for details.
