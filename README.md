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

The game requested several Vulkan device features that MoltenVK reported as unavailable on Apple Silicon.

## Workaround

The patch modifies MoltenVK to:

* expose vertex, geometry and tessellation evaluation subgroup stages;
* spoof the reported GPU vendor ID as AMD (`0x1002`);
* expose several Vulkan device features required by DOOM Eternal:

  * `geometryShader`
  * `depthBounds`
  * `wideLines`
  * `shaderCullDistance`

The patch also contains the debug logging used while investigating the device feature requests.

## Result

With the unmodified MoltenVK build, DOOM Eternal failed during `vkCreateDevice()` with `VK_ERROR_FEATURE_NOT_PRESENT`.

With the patched build:

* `vkCreateDevice()` succeeds;
* DOOM Eternal reaches the main menu;
* Vulkan swapchains are created successfully.

This is an **experimental compatibility workaround**, not an implementation of the missing Vulkan features in Metal. It has only been tested on the configuration listed above.

## Files

* `doom-eternal-moltenvk.patch` — patch against MoltenVK 1.4.3
* `libMoltenVK.dylib.zip` — prebuilt patched MoltenVK library

## Upstream issue

Related MoltenVK issue:

https://github.com/KhronosGroup/MoltenVK/issues/2830

## Disclaimer

This workaround may not work on other Apple Silicon GPUs, macOS versions, MoltenVK versions, CrossOver versions, or DOOM Eternal configurations.

Use the prebuilt library and patch at your own risk.
