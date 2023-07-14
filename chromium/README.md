# Chromium
This folder contains a patch for Chromium to compile with OpenXR support under Linux, loosely based on [this PR](https://github.com/chromium/chromium/pull/95).

# How to compile
In order to apply and compile this patch, it's important that you are able to compile Chromium yourself. For this follow the steps over at: https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md

Once you have the Chromium code base locally, apply the patch using the the following command (or use `git apply`):

```
git am /path/to/webxr-linux/chromium/webxr-linux-vulkan-2copy.patch
```

> **Note** These patches were made on top of Chromium 117, commit `d84f1a0ebf4e90528407c3e8a606a7b57f336a36`, and hopefully apply cleanly to more recent commits as well.

Once applied, build using `autoninja`. In order for chromium to find your configured OpenXR runtime make sure to launch it with the `--no-sandbox` flag
```
./chrome --no-sandbox
```

## Other relevant things to mention
 * Swapchain resizing is not implemented, but should be relatively trivial to add.
 * A copy of [this patch](https://groups.google.com/a/chromium.org/g/ozone-reviews/c/W8KlDt40SQY) is included to fix an issue preventing texture sharing.
 * Two texture copies are involved in the frame transfer: an ANGLE->Vulkan copy in the GPU process and a Vulkan->OpenXR copy in the XR process.
    * A call to `GLES2Interface::Finish()` is used to synchronize these two copies. A GPU-side semaphore would offer significantly reduced frametimes, but Chromium doesn't seem to offer a working IPC interface for them under Linux (the provided `gfx::GpuFence` interface is only implemented under Windows and Android).
    * The second copy is needed due to OpenXR's swapchain textures not being suitable for inter-process sharing.
 * There are different ways for the transport to take place, either `SUBMIT_AS_TEXTURE_HANDLE`, `SUBMIT_AS_MAILBOX_HOLDER` or `DRAW_INTO_TEXTURE_MAILBOX`.
    * This patch exclusively uses the `SUBMIT_AS_MAILBOX_HOLDER` mode. A future patch adding `DRAW_INTO_TEXTURE_MAILBOX` support could allow omitting a texture copy, improving performance.
 * As of July 2023, SteamVR's OpenXR implementation has a bug which deadlocks the process on instance destroy or exit, preventing subsequent WebXR sessions from starting after the first one finishes. See [this issue](https://github.com/ValveSoftware/SteamVR-for-Linux/issues/422) for more details.
