# Building the PicoGK Native Runtime for Linux

This document describes how to compile the native C++ runtime library for Linux x64 and integrate it with the PicoGK C# wrapper.

The C++ source lives in a separate repository: https://github.com/leap71/PicoGKRuntime

---

## Using a pre-built binary (Fedora only)

The `native/linux-x64/libpicogk.1.7.so` included in this repo was compiled on **Fedora 44**. It is dynamically linked against Fedora-versioned system libraries. On Fedora you can clone this repo, install the runtime dependencies below, and go — no compiler needed.

```bash
sudo dnf install -y tbb blosc lz4 libzstd zlib boost-iostreams boost-regex snappy
dotnet build
```

On other distributions (Ubuntu, Debian, Arch, etc.) the shared library version numbers will differ and the included binary will fail to load. In that case, follow the full build steps below to compile a native binary for your distro.

---

## Full build: compile from source

### Prerequisites

Tested on: **Fedora 44, x86_64, GCC 16, .NET SDK 9.0**

Install system build dependencies:

```bash
sudo dnf install -y \
  cmake gcc-c++ \
  boost-devel boost-iostreams \
  openvdb-devel tbb-devel blosc-devel \
  lz4-devel libzstd-devel zlib-devel xz-devel \
  wayland-devel wayland-protocols-devel libxkbcommon-devel \
  glfw-devel mesa-libGL-devel libXrandr-devel \
  libXcursor-devel libXinerama-devel libXi-devel
```

> **Note:** The Viewer component requires a display with OpenGL (X11 or Wayland). For headless geometry work, use the `Library` headless constructor — `Library.Go()` requires a display.

### 1. Clone and build PicoGKRuntime

```bash
git clone --recursive https://github.com/leap71/PicoGKRuntime
cd PicoGKRuntime
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
```

The compiled library is produced at:

```
build/lib/picogk.so
```

### 2. Replace the library in PicoGK

```bash
cp /path/to/PicoGKRuntime/build/lib/picogk.so \
   /path/to/PicoGK/native/linux-x64/libpicogk.1.7.so
```

### 3. Build and verify

```bash
dotnet build
```

A successful build copies `libpicogk.1.7.so` to `bin/Debug/net9.0/` alongside `PicoGK.dll`. The NuGet package (`dotnet pack`) includes the library under `runtimes/linux-x64/native/` for automatic resolution in consumer projects.

---

## How it integrates

| Scenario | How the library is found |
|---|---|
| Source build (`dotnet build`) | MSBuild copies it flat into the output directory |
| NuGet consumer (`dotnet add package PicoGK`) | .NET's RID resolver picks it from `runtimes/linux-x64/native/` |
| Custom location | Set `Config.strPicoGKLib` to the full absolute path including extension |
