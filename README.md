# DOOM Eternal — MoltenVK compatibility workaround

Experimental compatibility workaround for running **DOOM Eternal** on Apple Silicon through **MoltenVK** and **CrossOver**.

## Tested configuration

* MacBook Pro with Apple M4 Pro
* macOS 26.7
* CrossOver 26.3.0.39832
* MoltenVK 1.4.3
* DOOM Eternal

## Original problem

DOOM Eternal initially failed during Vulkan device initialization with:

```text
FATAL ERROR: Please update your driver:
Could not find support for required subgroup stages
```

After addressing the subgroup-stage check, `vkCreateDevice()` still failed with:

```text
VK_ERROR_FEATURE_NOT_PRESENT
```

MoltenVK reported several Vulkan device features requested by DOOM Eternal as unavailable on the Apple GPU.

The requested features observed during debugging included:

* `geometryShader`
* `depthBounds`
* `wideLines`
* `shaderCullDistance`

## Workaround

The patch modifies MoltenVK in several places to work around the game's Vulkan capability checks.

### Subgroup stages

The reported Vulkan subgroup stages are expanded from the native MoltenVK configuration:

```text
FRAGMENT | COMPUTE
```

to:

```text
VERTEX | GEOMETRY | TESSELLATION_EVALUATION | FRAGMENT | COMPUTE
```

### GPU vendor ID

The Apple GPU vendor ID reported by MoltenVK is spoofed as AMD:

```text
0x1002
```

This is required to get DOOM Eternal past its device initialization checks.

The driver version itself is **not spoofed** and remains the MoltenVK 1.4.3 version.

### Vulkan device features

The following Vulkan features are exposed to the game:

```text
geometryShader
depthBounds
wideLines
shaderCullDistance
```

These changes are compatibility workarounds for the game's feature checks. They should **not** be interpreted as proof that the underlying Apple Metal GPU natively provides the corresponding Vulkan functionality in the same way as a real Vulkan GPU.

## Result

With the unmodified MoltenVK build, DOOM Eternal failed during `vkCreateDevice()` with:

```text
VK_ERROR_FEATURE_NOT_PRESENT
```

The failure specifically corresponded to requested device features that MoltenVK reported as unavailable.

With the patched build:

* `vkCreateDevice()` succeeds;
* DOOM Eternal reaches the main menu;
* Vulkan swapchains are created successfully.

For example, MoltenVK reported successful creation of swapchains with sizes including:

```text
1512 x 982
1504 x 830
```

## Known limitations

This workaround does not provide full native support for every Vulkan feature that DOOM Eternal may use.

During testing, MoltenVK also reported a shader compilation failure for a shader using double-precision operations:

```text
Shader library compile failed
error: 'double' is not supported in Metal
```

The game continued running after this particular error during testing, but this demonstrates that the workaround does not eliminate all Vulkan/Metal compatibility limitations.

There were also Wine/CrossOver input-related warnings during testing, including `rawinput` warnings. These were observed but were not established as a MoltenVK compatibility failure.

## Debugging

The patch contains debug logging added while investigating the Vulkan feature requests made by DOOM Eternal.

This includes logging for:

* reported physical-device features;
* requested features passed to `vkCreateDevice()`;
* feature initialization.

The debug code is intentionally retained in the published patch because it documents the investigation and can help reproduce or extend the workaround.

## Files

* `doom-eternal-moltenvk.patch` — patch against MoltenVK 1.4.3
* `libMoltenVK.dylib.zip` — prebuilt patched MoltenVK library

## Upstream issue

Related MoltenVK issue:

[KhronosGroup/MoltenVK#2830](https://github.com/KhronosGroup/MoltenVK/issues/2830)

## Disclaimer

This is an **experimental compatibility workaround**.

It has only been tested on:

* Apple M4 Pro
* macOS 26.7
* CrossOver 26.3.0.39832
* MoltenVK 1.4.3

It may not work on other Apple Silicon GPUs, macOS versions, MoltenVK versions, CrossOver versions, or DOOM Eternal configurations.

The reported AMD vendor ID and exposed Vulkan features are compatibility spoofing/workarounds and do not imply that the Apple GPU actually implements the corresponding AMD/Vulkan capabilities natively.

Use the prebuilt library and patch at your own risk.
