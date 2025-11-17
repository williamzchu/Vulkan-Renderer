# Vulkan Renderer

A Vulkan-based rendering engine with comprehensive documentation available at [wzchu.com/notes/vulkan](https://wzchu.com/notes/vulkan).

## Prerequisites

Before building this project, you'll need to install the following tools:

- **[vcpkg](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started)** - C++ package manager
- **[CMake](https://cmake.org)** - Cross-platform build system generator

## Build Instructions

### 1. Install vcpkg

Follow the [official vcpkg installation guide](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started) to set up vcpkg on your system.

### 2. Install CMake

Download and install CMake from the [official website](https://cmake.org).

### 3. Configure the Project

Navigate to the project root directory and run the following command:

```bash
cmake -B build -S . -DVCPKG_TARGET_TRIPLET=x64-windows -DCMAKE_TOOLCHAIN_FILE=C:/PATH_TO_VCPKG/buildsystems/vcpkg.cmake
```

**Important:** Replace `C:/PATH_TO_VCPKG` with the actual path to your vcpkg installation directory.

### 4. Build the Project

After configuration completes successfully, build the project:

```bash
cmake --build build
```

## Documentation

For detailed implementation notes and technical documentation, visit [wzchu.com/notes/vulkan](https://wzchu.com/notes/vulkan).

## Platform Support

Currently configured for Windows (x64). To target other platforms, modify the `VCPKG_TARGET_TRIPLET` parameter accordingly.
