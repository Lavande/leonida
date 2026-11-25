# Windows 11 Build Troubleshooting & Source Verification

This page collects the most common setup issues encountered when running `Check-PC.bat`, `Install-Prerequisites.bat`, or `Build.bat` on Windows 10/11, along with a quick checklist for verifying that you are building from clean source.

## 1. Python 3.12+ Not Detected by `Build.bat`

`Build.bat` searches for Python 3.12+ via `py -3.14`/`3.13`/`3.12`/`3`, `where python`, and per-user installs under `%LOCALAPPDATA%\Programs\Python\Python3*`.

- **Symptom**: `Python 3.12 or newer is needed` even though Python is installed.
- **Fixes**:
  1. Re-run the Python 3.12+ installer from [python.org](https://www.python.org/downloads/) and tick **Add python.exe to PATH** and **py launcher**.
  2. Disable Windows Store App Execution Aliases for `python.exe` and `python3.exe` (**Settings → Apps → Advanced app settings → App execution aliases**).
  3. Open a fresh Command Prompt or PowerShell window so the updated `PATH` takes effect.

## 2. Prerequisite Installer (`scripts\install_prerequisites.ps1`) Blocked or Skipped

- `Build.bat` invokes `scripts\install_prerequisites.ps1 -Auto -Quiet` unless `LEONIDA_SKIP_PREREQ_INSTALL` is set.
- If you already installed the toolchain manually (Python 3.12+, .NET 8 SDK, Vulkan SDK, LLVM with AMDGPU target, Visual Studio 2022 C++ tools, CMake) and want `Build.bat` to skip prerequisite checks, run:
  ```cmd
  set LEONIDA_SKIP_PREREQ_INSTALL=1
  Build.bat
  ```
- Run `Check-PC.bat` first if you want a dry-run readiness check without starting `make_game.py`.

## 3. Missing `tools/rpf9dump` Submodule

- **Symptom**: Archive extraction or `rpf9dump` build fails with missing source files.
- **Fix**: Ensure submodules are initialized:
  ```cmd
  git submodule update --init --recursive
  ```

## 4. `spirv-val` or LLVM AMDGPU Tools Not Found on `PATH`

- Shader translation and SPIR-V validation require the LunarG Vulkan SDK (`spirv-val`) and LLVM built with the `AMDGPU` target.
- Verify in a new terminal:
  ```cmd
  spirv-val --version
  llvm-objdump --version
  ```
- If either command is missing, add `%VULKAN_SDK%\Bin` and your LLVM `bin` directory to your user `PATH` and reopen the terminal.

## 5. Insufficient Free SSD Space

- Extracting assets and translating all 26,503 shaders requires roughly **350 GB of free SSD space** (NVMe recommended).
- To place build outputs on a secondary drive instead of `Documents\Leonida`, pass `--out`:
  ```cmd
  Build.bat --out D:\Games\Leonida
  ```

## 6. Safe Source Verification (Avoiding Fake `.exe` Scams)

This repository distributes **GPLv3 source code only** and ships **no pre-compiled executables, keys, or game files**.

Before running any build script from a mirror or fork:
1. Never run pre-compiled `.exe` / `.msi` installers or password-protected archives (such as password `leonida`) claiming to be a playable GTA VI PC build.
2. Inspect `Build.bat`, `Install-Prerequisites.bat`, and `.gitmodules` in a plain-text editor to verify that all URLs point to legitimate upstream toolchains.
3. Keep all generated shaders, extracted assets, and saves local on your PC; never commit or upload them.

## Further Reading

- Illustrated Windows 11 setup & troubleshooting walkthrough: [gtaleonida.net/leonida-build-tutorial](https://gtaleonida.net/leonida-build-tutorial)
- Safe source audit & malware indicators checklist: [gtaleonida.net/project-leonida-safe-download](https://gtaleonida.net/project-leonida-safe-download)
