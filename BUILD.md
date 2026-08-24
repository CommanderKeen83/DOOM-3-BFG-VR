# Building DOOM 3 BFG VR (Fully Possessed)

This guide documents the modern build instructions and troubleshooting steps for compiling **DOOM 3 BFG VR** on Windows with modern Visual Studio (2017, 2019, or 2022) and CMake.

---

## 1. Prerequisites

Before building, make sure you have the following installed:

1. **Visual Studio 2017, 2019, or 2022**
   - Install the **Desktop development with C++** workload from the Visual Studio Installer.
2. **CMake (>= 3.10)**
   - Download from [cmake.org](https://cmake.org/download/) or install via winget/Chocolatey.
   - Ensure CMake is added to your system `PATH`.
3. **Microsoft DirectX SDK (June 2010)**
   - Download: [DirectX SDK (June 2010)](https://www.microsoft.com/en-us/download/details.aspx?id=6812)
   - Required for legacy audio and input headers (`d3d9.h`, `dinput8`, `x3daudio`, etc.).
   - Sets the `%DXSDK_DIR%` system environment variable.
4. **DOOM 3 BFG Edition (Retail Game)**
   - Required for the original game data and assets (from Steam, GOG, or retail disk).

---

## 2. Cloning & Submodules

Ensure all git submodules (especially **OpenVR**) are fetched:

```powershell
# In the DOOM-3-BFG-VR repository root:
git submodule update --init --recursive
```

---

## 3. Configuration & CMake Options

In `neo/CMakeLists.txt`, ensure your VR backend is configured:

- **SteamVR / OpenVR (Recommended for most headsets):**
  Set `option(OVR "Use Oculus VR's LibOVR SDK" OFF)` in `neo/CMakeLists.txt` or pass `-DOVR=OFF` during CMake configuration. This builds using OpenVR, supporting Meta Quest, Valve Index, HTC Vive, and Windows Mixed Reality through SteamVR.
- **Native Oculus SDK (Optional):**
  If building with native Oculus LibOVR (`OVR=ON`), download the [Oculus PC SDK 1.17.0](https://developer.oculus.com/downloads/package/oculus-sdk-for-windows/1.17.0/) and extract the `LibOVR` folder into `neo/libs/LibOVR`.

---

## 4. Generate the Build Files

Open PowerShell (or Command Prompt) in the repository root and generate the Visual Studio solution:

### Visual Studio 2022:
```powershell
cmake -B build -S neo -G "Visual Studio 17 2022" -A x64
```

### Visual Studio 2019:
```powershell
cmake -B build -S neo -G "Visual Studio 16 2019" -A x64
```

### Visual Studio 2017:
```powershell
cmake -B build -S neo -G "Visual Studio 15 2017" -A x64
```

---

## 5. Compiling

### Command Line:
```powershell
cmake --build build --config Release
```

### Visual Studio IDE:
1. Open `build/Doom3BFGVR.sln` in Visual Studio.
2. Set configuration to **Release** and platform to **x64**.
3. Build the solution (`Ctrl + Shift + B` or Build > Build Solution).

The generated executable will be placed in `build/Release/Doom3BFGVR.exe`.

---

## 6. Running & Testing

1. **Copy VR Assets:**
   Copy the `Fully Possessed` folder from `vr_assets/Fully Possessed` to your retail **DOOM 3 BFG Edition** directory:
   ```text
   C:\Program Files (x86)\Steam\steamapps\common\DOOM 3 BFG Edition\Fully Possessed
   ```

2. **Launch SteamVR / Oculus:**
   Ensure SteamVR is running and your headset and motion controllers are connected.

3. **Launch the Game:**
   Make sure runtime DLLs (FFmpeg, OpenAL, etc.) are present next to `Doom3BFGVR.exe`. You can either:
   - **Option A (Recommended):** Copy your compiled `build\Release\Doom3BFGVR.exe` directly into your DOOM 3 BFG Edition directory (where all DLLs and base game files already exist) and run it.
   - **Option B:** Copy the DLLs from your DOOM 3 BFG Edition directory into `build\Release\` and run:
     ```powershell
     .\build\Release\Doom3BFGVR.exe +set fs_basepath "C:\Program Files (x86)\Steam\steamapps\common\DOOM 3 BFG Edition"
     ```

---

## 7. Troubleshooting

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| Game silently exits immediately when launched | Missing runtime DLLs (`avcodec-58.dll`, `swscale-5.dll`, `OpenAL32.dll`, etc.). | Copy the runtime DLLs to `build\Release\` or copy `Doom3BFGVR.exe` directly into your DOOM 3 BFG Edition folder. |
| `Could NOT find DirectX (missing: DirectX_INCLUDE_DIR...)` | DirectX SDK (June 2010) is not installed or terminal hasn't refreshed environment variables. | Install DirectX SDK (June 2010) and restart the terminal so `%DXSDK_DIR%` is recognized. |
| `Cannot open include file: '../libs/LibOVR/Include/OVR_CAPI.h'` | `OVR` option is enabled without LibOVR SDK installed. | Set `OVR` to `OFF` in `neo/CMakeLists.txt` or configure with `-DOVR=OFF`. |
| `Compatibility with CMake < 3.5 has been removed` | Modern CMake requires `cmake_minimum_required(VERSION 3.5)`. | Ensure `neo/CMakeLists.txt` specifies `cmake_minimum_required(VERSION 3.5)`. |
| `Generator Ninja does not support platform specification...` | CMake defaulted to Ninja instead of Visual Studio. | Specify the generator explicitly with `-G "Visual Studio 17 2022"`. |
