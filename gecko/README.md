# Firefox
This folder contains a patch for firefox to allow WebXR content to work under Linux. It is more a hack than a proper implementation, and is only tested under X11, using a Valve Index as HMD.

# How to compile
In order to apply and compile this patch, it's important that you are able to compile Firefox yourself. For this follow the steps over at: https://firefox-source-docs.mozilla.org/setup/linux_build.html

> **Note** Don't make use of 'Artiface Mode' as the patch modifies the C++ code

Once you have the Firefox code base locally, apply the patch using the following command:

```
hg patch --no-commit path/to/webxr-linux.patch
```

Once applied, build using `./mach build` and if everything goes well, you should now have a built version with WebXR support. Run it using `./mach run`. There are some settings you will have to enable in `about:config`:
| Preference name | Value |
|-----------------|-------|
| `dom.vr.enabled` | true |
| `dom.vr.openvr.enabled` | true |
| `dom.vr.webxr.enabled`  | true |
| `dom.webvr`     | true  |

You are now ready to go and visit any WebXR experience on the web. Examples:
 * https://mixedreality.mozilla.org/hello-webxr/
 * https://immersive-web.github.io/webxr-samples/report/

# How it works
Firefox already contains all the relevant logic for handling the JS side of the WebXR specification and supports the OpenVR (Steam specific) runtime. Since SteamVR also works under Linux, the only missing part is actually submitting the frame data to the VRCompositor. It's sadly also this part that is difficult to implement cleanly. The rendering using WebGL is done using EGL (OpenGL ES 2.0), but to the best of my knowledge OpenVR only supports 'Desktop GL'.

To workaround this, the patch basically makes use of `glReadPixels` to load the raw pixel data from the OpenGL ES framebuffer and reuploads it as a new texture in a GLX context _every frame_. As a result, this solution isn't very performant, but when reducing the render resolution it's possible to hit 90HZ with relative ease.