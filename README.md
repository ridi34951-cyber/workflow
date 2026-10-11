# GyroLock V28 — Vulkan Native (Experimental)

V28 is an experimental migration of GyroLock's video stabilizer from OpenGL ES/EGL to a native Vulkan renderer. The Java stabilizer no longer imports or calls `android.opengl`, `GLES20`, `EGL14`, or `SurfaceTexture`. Rendering is performed by `app/src/main/cpp/gyrolock_vulkan.cpp` through JNI and Vulkan.

## Pipeline

1. `MediaExtractor` reads the original video track.
2. `MediaCodec` decodes frames to CPU-readable `YUV_420_888` images.
3. Java packs the decoder Y, U, and V planes into contiguous buffers; Vulkan performs YUV-to-RGB conversion in the fragment shader.
4. Vulkan uploads the RGBA frame to a GPU image, applies orientation/correction/crop/mirror-edge blur in a fragment shader, and presents the rendered frame to the MediaCodec encoder input surface.
5. `MediaCodec` encodes H.264 at a 30 FPS target.
6. The original audio track is remuxed without re-encoding.

The gyro correction arrays still come from the existing 240-point stabilization path. This version is experimental: Copying the decoded YUV planes through Java still adds CPU work; actual GPU/codec compatibility, frame timing, audio sync, portrait orientation, and output quality must be tested on a device. Vulkan support is required at runtime; unsupported devices should show a renderer error rather than silently falling back to OpenGL.

## Build on GitHub Actions

1. Upload the **contents** of this ZIP to the repository root.
2. Open **Actions** → **Build GyroLock Vulkan APK** → **Run workflow**.
3. Download the artifact named `GyroLock-V28-Vulkan-debug-apk` after a successful build.

The workflow installs Android NDK 27.2, CMake 3.22.1, and `glslang-tools`. GLSL shaders are compiled to SPIR-V as part of the Gradle pre-build task.

## Important limitations

- V28 has not been compiled or device-tested in the author's local environment; the GitHub Actions result is the first build check.
- `MediaCodec` decoder support for readable flexible YUV output varies by codec/device.
- The renderer requires a Vulkan-capable Android device and a Vulkan present queue that supports the encoder input surface.
- The current migration uses a genuine Vulkan-only render path, with YUV color conversion on the GPU, but speed is not guaranteed. Benchmarking and further optimization are still needed before treating it as a stable replacement for V27.
