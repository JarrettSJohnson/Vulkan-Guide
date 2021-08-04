# Atomics

The purpose of this chapter is to help users understand the various features Vulkan exposes for atomic operations.

# Variations of Atomics

To better understand the different extensions, it is first important to be aware of the various types of atomics exposed.

- Type
    - `float`
    - `int`
- Width
    - `16 bit`
    - `32 bit`
    - `64 bit`
- Operations
    - loads
    - stores
    - exchange
    - add
    - min
    - max
    - etc.
- Storage Class
    - `StorageBuffer` or `Uniform` (buffer)
    - `Workgroup` (shared memory)
    - `Image` (image or sparse image)

# Baseline Support

With Vulkan 1.0 and no extensions, an application is allowed to use `32-bit int` type for atomics. This can be used for all supported SPIR-V operations (load, store, exchange, etc). SPIR-V contains some atomic operations that are guarded with the `Kernel` capability and are not currently allowed in Vulkan.

## Atomic Counters

While both GLSL and SPIR-V support the use of atomic counters, Vulkan does **not** expose the `AtomicStorage` SPIR-V capability needed to use the `AtomicCounter` storage class. It was decided that an app can just use `OpAtomicIAdd` and `OpAtomicISub` with the value `1` to achieve the same results.

## Expanding Atomic support

The current extensions that expose additional support for atomics are:

- [VK_KHR_shader_atomic_int64](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VK_KHR_shader_atomic_int64.html)
- [VK_EXT_shader_image_atomic_int64](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VK_EXT_shader_image_atomic_int64.html)
- [VK_EXT_shader_atomic_float](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VK_EXT_shader_atomic_float.html)
- [VK_EXT_shader_atomic_float2](https://www.khronos.org/registry/vulkan/specs/1.2-extensions/man/html/VK_EXT_shader_atomic_float2.html)

Each explained in more details below.

# VK_KHR_shader_atomic_int64

> Promoted to core in Vulkan 1.2
>
> [GLSL - GL_EXT_shader_atomic_int64](https://github.com/KhronosGroup/GLSL/blob/master/extensions/ext/GL_EXT_shader_atomic_int64.txt)

This extension allows for `64-bit int` atomic operations for **buffers** and **shared memory**. If the `Int64Atomics` SPIR-V capability is declared, all supported SPIR-V operations can be used with `64-bit int`.

The two feature bits, `shaderBufferInt64Atomics` and `shaderSharedInt64Atomics`, are used to query what storage classes are supported for `64-bit int` atomics.

- `shaderBufferInt64Atomics` - buffers
- `shaderSharedInt64Atomics` - shared memory

The `shaderBufferInt64Atomics` is always guaranteed to be supported if using Vulkan 1.2 or the extension is exposed.

# VK_EXT_shader_image_atomic_int64

> [SPV_EXT_shader_image_int64](https://htmlpreview.github.io/?https://github.com/KhronosGroup/SPIRV-Registry/blob/master/extensions/EXT/SPV_EXT_shader_image_int64.html)
>
> [GLSL_EXT_shader_image_int64](https://github.com/KhronosGroup/GLSL/blob/master/extensions/ext/GLSL_EXT_shader_image_int64.txt)

This extension allows for `64-bit int` atomic operations for **images** and **sparse images**. If the `Int64Atomics` and `Int64ImageEXT` SPIR-V capability is declared, all supported SPIR-V operations can be used with `64-bit int` on images.

## Image vs Sparse Image support

This extension exposes both a `shaderImageInt64Atomics` and `sparseImageInt64Atomics` feature bit. The `sparseImage*` feature is an additional feature bit and is only allowed to be used if the `shaderImage*` bit is enabled as well. Some hardware has a hard time doing atomics on images with [sparse resources](./sparse_resources.md), therefor the atomic feature is split up to allow **sparse images** as an additional feature an implementation can expose.

# VK_EXT_shader_atomic_float

> [SPV_EXT_shader_atomic_float_add](https://htmlpreview.github.io/?https://github.com/KhronosGroup/SPIRV-Registry/blob/master/extensions/EXT/SPV_EXT_shader_atomic_float_add.html)
>
> [GLSL_EXT_shader_atomic_float](https://github.com/KhronosGroup/GLSL/blob/master/extensions/ext/GLSL_EXT_shader_atomic_float.txt)

This extension allows for `float` atomic operations for **buffers**, **shared memory**, **images**, and **sparse images**. Only a subset of operations is supported for `float` types with this extension.

The extension lists many feature bits. One way to group them is by `*Float*Atomics` and `*Float*AtomicAdd`:

- The `*Float*Atomics` features allow for the use of `OpAtomicStore`, `OpAtomicLoad`, and `OpAtomicExchange` for `float` types.
    - Note the `OpAtomicCompareExchange` "exchange" operation is not included as the SPIR-V spec only allows `int` types for it.
- The `*Float*AtomicAdd` features allow the use of the two extended SPIR-V operations `AtomicFloat32AddEXT` and `AtomicFloat64AddEXT`.

From here the rest of the permutations of features fall into the grouping of `32-bit float` support:
- `shaderBufferFloat32*` - buffers
- `shaderSharedFloat32*` - shared memory
- `shaderImageFloat32*` - images
- `sparseImageFloat32*` - sparse images

and `64-bit float` support:
- `shaderBufferFloat64*` - buffers
- `shaderSharedFloat64*` - shared memory

> Note: OpenGLES [OES_shader_image_atomic](https://www.khronos.org/registry/OpenGL/extensions/OES/OES_shader_image_atomic.txt) allowed the use of atomics on `r32f` for `imageAtomicExchange`. For porting, an application will want to check for `shaderBufferFloat32Atomics` support to be able to do the same in Vulkan.

# VK_EXT_shader_atomic_float2

> [SPV_EXT_shader_atomic_float_min_max](https://htmlpreview.github.io/?https://github.com/KhronosGroup/SPIRV-Registry/blob/master/extensions/EXT/SPV_EXT_shader_atomic_float_min_max.html)
>
> [SPV_EXT_shader_atomic_float16_add](https://htmlpreview.github.io/?https://github.com/KhronosGroup/SPIRV-Registry/blob/master/extensions/EXT/SPV_EXT_shader_atomic_float16_add.html)
>
> [GLSL_EXT_shader_atomic_float](https://github.com/KhronosGroup/GLSL/blob/master/extensions/ext/GLSL_EXT_shader_atomic_float.txt)

This extension adds 2 additional sets of features missing in `VK_EXT_shader_atomic_float`

First, it adds `16-bit floats` for both **buffers** and **shared memory** in the same fashion as found above for `VK_EXT_shader_atomic_float`.

- `shaderBufferFloat16*` - buffers
- `shaderSharedFloat16*` - shared memory

Second, it adds `float` support for `min` and `max` atomic operations (`OpAtomicFMinEXT` and `OpAtomicFMaxEXT`)

For `16-bit float` support (with `AtomicFloat16MinMaxEXT` capability):
- `shaderBufferFloat16AtomicMinMax` - buffers
- `shaderSharedFloat16AtomicMinMax` - shared memory

For `32-bit float` support (with `AtomicFloat32MinMaxEXT` capability):
- `shaderBufferFloat32AtomicMinMax` - buffers
- `shaderSharedFloat32AtomicMinMax` - shared memory
- `shaderImageFloat32AtomicMinMax` - images
- `sparseImageFloat32AtomicMinMax` - sparse images

For `64-bit float` support (with `AtomicFloat64MinMaxEXT` capability):
- `shaderBufferFloat64AtomicMinMax` - buffers
- `shaderSharedFloat64AtomicMinMax` - shared memory