# GE Engine Sandbox

## Overview

This repository is a sandbox environment for testing while developing [ge-engine](https://github.com/CallumFerguson/ge-engine). It can also be used as a template for creating a new ge-engine application.

GE Engine has a separate engine library and engine tools executables, but does not include an editor. For
example, the `PackageAssets` tool should be run before every build to package any new or changed assets.

This repository includes a `.run` folder with CLion run configurations to automate these tasks. If you don't use CLion,
you'll need to set up a build script that uses the `PackageAssets` tool or run `PackageAssets` manually whenever you
modify assets.

## Build Instructions

### CLion Setup

1. **Clone the Repository:**

   ```sh
   git clone https://github.com/CallumFerguson/ge-engine-sandbox.git
   cd ge-engine-sandbox
   git submodule update --init
   ```

2. **Open ge-engine-sandbox in CLion.**

#### Native Build

1. **Configure CMake Toolchain:**
    - Select Visual Studio for the CMake Toolchain.

2. **Select Run Configuration:**
    - Choose the `App` run configuration.

3. **Build or Run:**
    - Press build or run.

CMake will automatically download and build the Dawn repository, which may take some time. Using the provided run
configurations will ensure assets are automatically packaged before each run.

#### Emscripten Build

1. **Prerequisite:**
    - Perform a native build first since the `PackageAssets` tool must run natively to package assets for the Emscripten
      build. The run configurations will automatically package assets before each run using a Release build
      of `PackageAssets`.

2. **Create Emscripten Toolchain:**
    - Create a System Toolchain called Emscripten.
    - Select `emcc` and `em++` for the C and C++ compilers:
        - `path\to\emsdk\upstream\emscripten\emcc.bat`
        - `path\to\emsdk\upstream\emscripten\em++.bat`

3. **Create CMake Profiles:**
    - Create profiles for Debug and Release builds.
    - Select the Emscripten toolchain.
    - Set the CMake Toolchain File option:
        - `-DCMAKE_TOOLCHAIN_FILE=path\to\emsdk\upstream\emscripten\cmake\Modules\Platform\Emscripten.cmake`

4. **Build Types:**
    - **Debug Build:** Compiles faster, includes more debugging information and logging but has poor performance.
    - **Release Build:** Compiles slower, creates smaller builds, and hides exceptions but offers better performance.
    - **Debug-Faster-Emscripten Build:** Create a third CMake profile using Debug and the additional CMake
      option `-DDEBUG_FASTER=ON` for a balance between compile speed, exception handling, and performance (uses
      optimization level O2).

5. **Edit Run Configuration:**
    - Edit the `App-Emscripten` run configuration.
    - Make sure that under "Before Launch" PackageAssets-NoCopy uses Release-Visual Studio CMake profile. The package assets tool cannot be built with emscsripten
      - it should look like "Run 'CMake Application 'PackageAssets-NoCopy' | Release-Visual Studio'"
    - Make sure that preloadAssetsEmscripten.bat is run after PackageAssets-NoCopy and before Build
      - Create or update the external Tool `External Tools/PreloadAssetsEmscripten`:
          - Program: `$ContentRoot$\App\platform\emscripten\scripts\preloadAssetsEmscripten.bat`
          - Arguments: `$CMakeCurrentBuildDir$ path\to\emsdk`
          - Working Directory: `$ContentRoot$\App`

6. **Build and run:**
    - Press run (only building will not trigger asset packing).
    - Note that I may crash on the first build because it will try to package assets into a folder that does not exist until after the first build. Just press run again.
      - TODO: fix this
