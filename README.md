# Float16Array

A proposal to add float16 (aka half-precision or binary16) TypedArrays to JavaScript.

## Status

Authors: Kevin Gibbons

Champions: Leo Balter

This proposal is at Stage 1 of [The TC39 Process](https://tc39.es/process-document/).

Spec text is available [here](https://tc39.es/proposal-float16array/).

## Motivation

- Explicit request from the Color on the Web CG, for [their use case](https://github.com/w3c/ColorWeb-CG/blob/main/canvas_float.md) with float-backed canvases.
- Useful for GPU operations, where full precision often isn't necessary and memory constraints are serious.
  - WebGPU [supports float16](https://github.com/gpuweb/gpuweb/issues/2512). It exposes/consumes raw ArrayBuffers. When those contain 32-bit floats, [the pattern](https://gpuweb.github.io/gpuweb/explainer/#example-eed014f1) is to wrap the buffer in `new Float32Array(...)`; that doesn't work when they contain 16-bit floats.
  - [Increasingly relevant](https://github.com/huggingface/blog/blob/main/stable_diffusion.md) with new tools like Stable Diffusion, where full-precision representations don't fit in VRAM on many machines.
- WebGL/WebGL2 support float16.
  - WebGL's `OES_texture_half_float` extension and WebGL2 [support float16 texture](https://registry.khronos.org/webgl/extensions/OES_texture_half_float/), and also both `EXT_color_buffer_half_float` extensions [support float16 render target](https://registry.khronos.org/webgl/extensions/EXT_color_buffer_half_float/). They expose/consume Uint16Arrays; that is not easy to handle.
- ARM and IA-32 and Intel® 64 architectures support float16 intrinsics.
  - In the C++23 specification, [`std::float16_t` will be supported](https://en.cppreference.com/w/cpp/types/floating-point) to call the CPU instructions.
- Faking it in userland has [serious performance costs](https://github.com/petamoriken/float16/issues/781).

## Proposal

This would add a new kind of TypedArray, `Float16Array`, to complement the existing `Float32Array` and `Float64Array`. It would also add two new methods on `DataView` for reading and setting float16 values, as `getFloat16` and `setFloat16`, to complement the existing similar methods for working with full and double precision floats.

## Userland

[@petramoriken](https://github.com/petamoriken) has [a package implementing `Float16Array`](https://github.com/petamoriken/float16) which gets 100k+ downloads/week [on npm](https://www.npmjs.com/package/@petamoriken/float16). See that repository for examples of some of the limitations and downsides of trying to do this purely in userland - notably the impossibility of integrating correctly with other web platform features like `WebGL`'s `HALF_FLOAT` buffers and `structuredClone`.
