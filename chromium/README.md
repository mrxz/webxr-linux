# Chromium
This folder contains a series of patches for Chromium to compile with OpenXR support under Linux. Chromium's implementation is tied to DirectX in several places. The first patch in the series is loosely based on [this PR](https://github.com/chromium/chromium/pull/95) to get OpenXR in chromium to compile under Linux. Further patches slowly fill in the gaps, and an `XrSession` can be made, just getting the image from the render process on the swapchain is lacking (= black screen, but HMD and controllers tracked and reported back to the WebXR app).

# How to compile
In order to apply and compile this patch, it's important that you are able to compile Chromium yourself. For this follow the steps over at: https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md

Once you have the Chromium code base locally, apply the series of patches using the the following command (or use `git apply`):

```
git am /path/to/webxr-linux/chromium/*.patch
```

> **Note** These patches were made on top of Chromium commit `64fc9d454056ea3806350053693d7d81220addcb` and hopefully apply cleanly to more recent commits as well.

Once applied, build using `autoninja`. In order for chromium to find your configured OpenXR runtime make sure to launch it with the `--no-sandbox` flag
```
./chrome --no-sandbox
```

## Other relevant things to mention
 * The code in `xr_frame_transport.cc` is relevant, check the `FrameSubmit` method.
 * There are different ways for the transport to take place, either `SUBMIT_AS_TEXTURE_HANDLE`, `SUBMIT_AS_MAILBOX_HOLDER` or `DRAW_INTO_TEXTURE_MAILBOX`.
    * The mode used by default (for Windows) is `SUBMIT_AS_TEXTURE_HANDLE`
    * The `DRAW_INTO_TEXTURE_MAILBOX` mode is also implemented (for Windows), but not active by default. Depends on if `IsUsingSharedImages` is true (`openxr_api_wrapper.cc`) which depends on the presence of `mailbox_holder` in the `colo_swapchain_images_` which are created in `OpenXrApiWrapper::CreateSharedMailboxes` depending on the outcome of `OpenXrApiWrapper::ShouldCreateSharedImages`
 * Chromium has support for `GpuMemoryBuffers` under Linux (though never shipped): https://bugs.chromium.org/p/chromium/issues/detail?id=1031269
    * This does require launching with both `--enable-native-gpu-memory-buffers` and `--use-gl=desktop`. Check the table under `GpuMemoryBuffers Status` under `chrome://gpu`. It should report the supported usages per format instead of just 'Software Only'.