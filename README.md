# MLV Decode Engine

A minimal, portable extraction of the decoding and raw-processing core from
[MLV-App](https://github.com/ilia3101/MLV-App) — no UI, no dependencies beyond
what the sources themselves require.

**Status:** Experimental. API surface and folder layout may change.

---

## What this is

[Magic Lantern](https://www.magiclantern.fm/) is a third-party firmware extension
for Canon cameras. It can record video in **MLV** (Magic Lantern Video) format —
a structured raw container carrying unprocessed sensor data, metadata blocks, and
optional audio. **MCRAW** is the analogous format used by MotionCam Pro.

This repository contains only the source folders needed to:

- Parse and decode MLV / MCRAW files
- Apply raw corrections (chromatic aberration, bad-pixel, lens corrections)
- Debayer raw sensor data to RGB
- Apply a color-grading pipeline (matrices, curves, tone mapping)

The goal is a single, self-contained source tree that can be dropped into an
Android NDK or iOS/Xcode project without dragging in the rest of MLV-App.

---

## Source layout

| Path | Purpose |
|------|---------|
| `mlv/` | MLV / MCRAW container parser — reads blocks, extracts frames and metadata |
| `processing/` | Top-level raw processing pipeline that coordinates the stages below |
| `debayer/` | Demosaicing algorithms (converts raw Bayer grid → RGB) |
| `librtprocess/` | Ported subset of [RawTherapee's](https://rawtherapee.com/) processing routines |
| `matrix/` | Color-matrix math — camera-native → XYZ → display color space |
| `ca_correct/` | Chromatic aberration correction |
| `basic_patches/` | Bad-pixel and hot-pixel patching |
| `dng/` | DNG writing support |
| `mlv_include.h` | Convenience umbrella header |

---

## Who this is for

- Developers building a **minimal MLV/MCRAW viewer or converter** without a full
  Qt/GUI dependency
- Mobile engineers porting MLV playback to **Android (NDK/JNI)** or **iOS
  (Xcode/Swift package)**
- Researchers or tinkerers who want to experiment with Canon raw data at the
  C/C++ level

---

## Building with CMake

Requires CMake 3.10+, a C99 compiler, and a C++11 compiler.

```sh
cmake -B build
cmake --build build
```

This produces **`libmlvcore.so`** (Linux/Android) or **`libmlvcore.dylib`** (macOS/iOS) —
a single shared library that bundles all static components below.

The exact list of compiled source files is in [`CMakeLists.txt`](CMakeLists.txt).

### Library breakdown

The build compiles six static libraries that are then linked into the final shared library:

| CMake target | Sources | Role |
|---|---|---|
| `mlv` | `mlv/` | MLV container parser, Lossless JPEG (lj92), low-level raw processing |
| `mcraw` | `mlv/mcraw/` | MotionCam Pro Raw decoder |
| `processing` | `processing/` | Raw pipeline — blur, denoiser, CA filter, spline, RBF, LUT, etc. |
| `debayer` | `debayer/` | Demosaicing (AHD, AMaZE, basic, convolution, SLEEF SIMD) |
| `ca` | `ca_correct/` | RawTherapee-derived chromatic aberration correction |
| `matrix` | `matrix/` | Color matrix transforms (camera-native → XYZ → display) |
| `dng` | `dng/` | DNG file writing |
| `rtprocess` | `librtprocess/` | Additional RawTherapee processing routines (built via `add_subdirectory`) |

### Include paths

The build exposes the following include directories:

```
./                        (for mlv_include.h)
avir/
ca_correct/
debayer/
dng/
matrix/
mlv/
mlv/mcraw/
processing/
librtprocess/src/include/
```

### Integrating into another CMake project

```cmake
add_subdirectory(mlv-decode-engine)
target_link_libraries(your_target PRIVATE mlvcore)
```

Or link `libmlvcore` manually and add the include paths listed above.

---

## License

This project inherits the license of
[MLV-App](https://github.com/ilia3101/MLV-App) —
**GNU General Public License v3.0**. See [LICENSE](LICENSE) or the upstream
repository for the full terms.

---

## Upstream

All core logic originates from **MLV-App** by Ilia Sibiryakov and contributors.
This repository is a focused extraction, not a fork with independent changes.
Please report issues that are clearly bugs in the original decoding logic
upstream at <https://github.com/ilia3101/MLV-App/issues>.
