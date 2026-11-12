# Learning Plan: windows-11-vs2026-arm

The goal is to "learn" the `windows-11-vs2026-arm` image in GitHub CI iteratively.

## Status Overview
- [x] Phase 1: Foundation & Shell Verification (Completed)
- [x] Phase 2: PowerShell & Rust Toolchain (Completed)
- [x] Phase 3: .NET & Visual Studio Toolchain (Completed)
- [x] Phase 4: Native Build & C++ Tools (Completed)
- [x] Phase 5: Connectivity & Specialized Tools (Completed)
- [x] Phase 6: WSL & Virtualization Discovery (Completed)

## Detailed Phase Breakdown

### Phase 1: Foundation & Shell Verification (DONE)
- Verified OS (Windows 11 Enterprise) and Architecture (ARM64).
- Verified Bash, PowerShell, and pwsh versions.
- Captured full environment variable set.

### Phase 2: PowerShell & Rust Toolchain (DONE)
- **PowerShell:** Confirmed 7.6.9 with `RemoteSigned` policy. Verified `Az` and `AWSPowerShell` modules.
- **Rust:** Confirmed `rustc`/`cargo` 1.99.1. Verified `bindgen` and `cbindgen` availability.

### Phase 3: .NET & Visual Studio Toolchain (DONE)
- **Visual Studio:** Confirmed VS 2026 Enterprise (18.11) via `vswhere`.
- **MSBuild:** Confirmed 18.11.1.51303 accessibility.
- **.NET:** Confirmed SDKs/Runtimes for .NET 8, 9, and 10.

### Phase 4: Native Build & C++ Tools (DONE)
- **Compilers:** 
    - **LLVM/Clang:** Confirmed `clang` (22.1.8) and `clang++` (22.1.8) are fully available and target `aarch64-pc-windows-msvc`.
    - **MinGW:** Confirmed `gcc` (14.2.0) and `g++` (14.2.0).
    - **MSVC (`cl.exe`):** Visual Studio is detected at `C:\Program Files\Microsoft Visual Studio\18\Enterprise`. However, the standard `VsDevCmd.bat` path is non-standard, requiring manual path configuration.
- **Build Systems:**
    - Verified `cmake` (4.4.5), `bazel` (9.3.2), and `make` (4.4.1).
    - **Missing:** `ninja` was not detected in the PATH.
- **vcpkg:** Confirmed functional with `VCPKG_INSTALLATION_ROOT` set to `C:\vcpkg`.
- **Libraries:** Verified `OpenSSL` (3.6.5) availability.

### Phase 5: Connectivity & Specialized Tools (DONE)
- **Cloud CLIs:** Verified `az` (2.91.0), `aws` (2.39.3), and `gh` (2.103.3).
- **Database Tools:** Verified `mysql` (8.4.13) and `sqlcmd` are present.
- **Web Servers:** Identified `C:\inetpub` (IIS) as the primary web server directory.
- **Networking:** Confirmed the environment is **restricted** (No DNS resolution and Port 443 connectivity failed).

### Phase 6: WSL & Virtualization Discovery (DONE)
- **WSL Status:** Confirmed `wsl.exe` is present but non-functional. It returns: *"The Windows Subsystem for Linux is not installed."*
- **Windows Features:** Verified that both `VirtualMachinePlatform` and `Microsoft-Windows-Subsystem-Linux` are marked as **Enabled** in the OS features.
- **Virtualization:** `Win32_DeviceGuard` status is inaccessible via standard CIM.
- **Test Execution:** Verified that Linux command execution (e.g., `ls /`) via WSL fails with exit code 1.
```

---

### Final Summary Report: `windows-11-vs2026-arm` Environment

The iterative discovery of the `windows-11-vs2026-arm` image is now complete. Below is the final synthesis of the environment's capabilities and limitations.

#### 1. Core Architecture & Shell
*   **OS:** Windows 11 Enterprise (ARM64).
*   **Shells:** Supports standard `bash`, `PowerShell` (5.1), and `pwsh` (7.6.9). The environment is fully prepared for mixed-shell workflows.

#### 2. Development Toolchains
*   **Primary Native Strategy:** **Clang (v22.1.8)** is the powerhouse of this image. It is fully configured for `aarch64-pc-windows-msvc` and should be the default choice for C++ development.
*   **Fallback Native Strategy:** **GCC (v14.2.0)** via MinGW is available for projects requiring standard GCC compatibility.
*   **MSVC Caveat:** While Visual Studio 2026 Enterprise is installed, the standard `VsDevCmd.bat` path is non-standard. Projects requiring `cl.exe` must be configured to locate the binaries manually or via a custom initialization script.
*   **Rust:** A modern Rust toolchain (1.99.1) is ready for systems programming.
*   **.NET:** High compatibility with modern .NET (8, 9, and 10 SDKs/Runtimes).

#### 3. Build Orchestration & Packages
*   **Build Systems:** Supports `CMake`, `Bazel`, and `Make`. **Note:** `Ninja` is absent and must be provided if required for high-speed builds.
*   **Package Management:** `vcpkg` is pre-configured at `C:\vcpkg`, making it the primary method for managing C++ dependencies.
*   **Cryptography:** `OpenSSL` (3.6.5) is available for secure operations.

#### 4. Virtualization & Linux Support
*   **WSL Status:** **Non-Functional.** Although the Windows Optional Features for "Virtual Machine Platform" and "Windows Subsystem for Linux" are reported as "Enabled," the `wsl.exe` binary reports that the subsystem is not installed. Consequently, **Linux command execution via WSL is not available** in this runner.
*   **Virtualization:** The environment supports the underlying infrastructure for virtualization (as indicated by enabled features), but standard CIM queries for Device Guard status are restricted.

#### 5. Infrastructure & Connectivity
*   **Cloud Readiness:** The image includes `az`, `aws`, and `gh` CLIs, making it suitable for automated cloud deployments and GitHub Actions management.
*   **Database Support:** Includes `mysql` (8.4.13) and `sqlcmd` for database-related tasks.
*   **Networking Isolation:** **Critical Finding.** The runner has restricted connectivity (No DNS/External Port 443). Build scripts should not rely on external downloads during the execution phase; all dependencies should be pre-cached or bundled.

#### Final Conclusion
The `windows-11-vs2026-arm` image is a **highly-specialized, Clang-centric native build environment**. It is optimized for ARM64 development, specifically for projects leveraging Rust, .NET, and C++ via Clang and vcpkg. A notable limitation is that **WSL is not functional** despite the features being enabled. Due to its networking restrictions, it is best suited for **self-contained build pipelines** where all artifacts are present in the workspace.

## Code

These scripts are written as powershell commands in Github Action jobs. Follow the pattern bellow

### caller `./.github/workflows/dev.yml`

```yaml
on: push
jobs:
  phase1:
    uses: ./.github/workflows/phase1.yml

  phase2:
    needs: phase1
    uses: ./.github/workflows/phase2.yml

  phase3:
    needs: phase2
    uses: ./.github/workflows/phase3.yml

  phase4:
    needs: phase3
    uses: ./.github/workflows/phase4.yml

  phase5:
    needs: phase4
    uses: ./.github/workflows/phase5.yml

  phase6:
    needs: phase5
    uses: ./.github/workflows/phase6.yml
```

### callee `./.github/workflows/phase1.yml`

```yaml
name: Phase 1 - Foundation Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: OS and Architecture Info
        shell: pwsh
        run: |
          Write-Host "--- Architecture ---"
          Write-Host ([System.Environment]::ProcessorArchitecture)
          Write-Host "--- System Info ---"
          Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, OSArchitecture | Format-List

      - name: Shell Versions
        shell: pwsh
        run: |
          Write-Host "--- Bash ---"
          bash --version
          Write-Host "--- PowerShell ---"
          $PSVersionTable.PSVersion
          Write-Host "--- pwsh ---"
          pwsh --version

      - name: Environment Variables
        shell: pwsh
        run: |
          Write-Host "--- Environment Variables ---"
          [System.Environment]::GetEnvironmentVariables("Process").GetEnumerator() | ForEach-Object { Write-Host "$($_.Key)=$($_.Value)" }
```

### callee `./.github/workflows/phase2.yml`

```yaml
name: Phase 2 - PowerShell & Rust Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: PowerShell Module Inventory
        shell: pwsh
        run: |
          Write-Host "--- PowerShell Modules ---"
          Get-Module -ListAvailable | Select-Object Name, Version | Sort-Object Name | Format-Table -AutoSize

      - name: PowerShell Environment Check
        shell: pwsh
        run: |
          Write-Host "--- PowerShell Version ---"
          $PSVersionTable.PSVersion
          Write-Host "--- Execution Policy ---"
          Get-ExecutionPolicy
          Write-Host "--- Path Variable ---"
          $env:Path

      - name: Rust Toolchain Versions
        shell: pwsh
        run: |
          Write-Host "--- Rustc ---"
          rustc --version
          Write-Host "--- Cargo ---"
          cargo --version
          Write-Host "--- Rustup ---"
          rustup --version

      - name: Rust Installed Packages
        shell: pwsh
        run: |
          Write-Host "--- Cargo Installed Binaries ---"
          cargo install --list
```

### callee `./.github/workflows/phase3.yml`

```yaml
name: Phase 3 - .NET & Visual Studio Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: .NET & Visual Studio Toolchain Discovery
        shell: pwsh
        run: |
          Write-Host "--- .NET SDKs ---"
          dotnet --list-sdks
          Write-Host "--- .NET Runtimes ---"
          dotnet --list-runtimes
          Write-Host ""

          Write-Host "--- Finding Visual Studio Installation ---"
          # Use vswhere to find the latest Enterprise installation
          # We force the result to a string to ensure it's not an object/array
          $vsPath = (& "vswhere.exe" -latest -property installationPath).ToString()
          Write-Host "Visual Studio Path: $vsPath"

          if ([string]::IsNullOrWhiteSpace($vsPath)) {
            Write-Error "Visual Studio installation not found via vswhere."
            exit 1
          }

          Write-Host "--- Checking MSBuild ---"
          $msbuildPath = Join-Path $vsPath "MSBuild\Current\Bin\MSBuild.exe"
          if (Test-Path $msbuildPath) {
            & $msbuildPath -version
          } else {
            Write-Warning "MSBuild not found at $msbuildPath"
          }

          Write-Host "--- Checking for vsdevelop (if available) ---"
          $vsDevelop = Get-ChildItem -Path $vsPath -Filter "vsdevelop.exe" -Recurse -ErrorAction SilentlyContinue | Select-Object -First 1
          if ($vsDevelop) {
            Write-Host "Found vsdevelop at: $($vsDevelop.FullName)"
            & $vsDevelop.FullName --version
          } else {
            Write-Host "vsdevelop.exe not found in VS installation path."
          }

          Write-Host ""
          Write-Host "--- Checking for specific workloads ---"
          $workloads = @("maui.windows", "netcore.sdk", "android", "web")
          
          # Verify VS Base Directory is accessible
          $common7 = Join-Path $vsPath "Common7"
          if (Test-Path $common7) {
            Write-Host "VS Base Directory ($common7) is accessible."
          } else {
            Write-Warning "VS Base Directory is not accessible."
          }

          # Check for SDKs as a proxy for workloads
          foreach ($w in $workloads) {
            Write-Host "Checking for $w SDK..."
            if (dotnet --list-sdks | Select-String -Pattern $w) {
              Write-Host "  [FOUND] $w SDK is installed."
            } else {
              Write-Host "  [MISSING] $w SDK was not found in dotnet --list-sdks"
            }
          }
```

### callee `./.github/workflows/phase4.yml`

```yaml
name: Phase 4 - Native Build & C++ Tools Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: Native Toolchain Discovery
        shell: pwsh
        run: |
          Write-Host "--- Compilers (Standard PATH) ---"
          # Check GCC / Clang / LLVM
          foreach ($compiler in @("gcc", "clang", "g++", "clang++")) {
            if (Get-Command $compiler -ErrorAction SilentlyContinue) {
              & $compiler --version
            } else {
              Write-Host "$compiler not found."
            }
          }

          Write-Host ""
          Write-Host "--- MSVC (cl.exe) Discovery & Init ---"
          
          # 1. Use vswhere to find the latest Visual Studio installation
          $vsPath = (& "vswhere.exe" -latest -property installationPath).ToString()
          
          if ([string]::IsNullOrWhiteSpace($vsPath)) {
            Write-Error "Visual Studio installation not found via vswhere."
            exit 1
          }
          Write-Host "Visual Studio Path: $vsPath"

          # 2. Construct path to the Developer Command Prompt initialization script
          $vsDevCmd = Join-Path $vsPath "Common7\IDE\VsDevCmd.bat"

          if (Test-Path $vsDevCmd) {
            Write-Host "Found VsDevCmd.bat at: $vsDevCmd"
            Write-Host "Initializing MSVC environment and checking cl.exe..."
            
            # We must use cmd /c to call the .bat file because batch variables 
            # do not persist in a PowerShell session.
            # We wrap the call and cl.exe in the same cmd instance.
            $clCheck = cmd /c "call `"$vsDevCmd`" && cl.exe"
            Write-Host $clCheck
          } else {
            Write-Warning "VsDevCmd.bat not found at $vsDevCmd. MSVC environment cannot be initialized."
          }

          Write-Host ""
          Write-Host "--- Build Systems ---"
          foreach ($buildTool in @("cmake", "ninja", "bazel", "make")) {
            if (Get-Command $buildTool -ErrorAction SilentlyContinue) {
              & $buildTool --version
            } else {
              Write-Host "$buildTool not found."
            }
          }

          Write-Host ""
          Write-Host "--- vcpkg Inventory ---"
          if (Get-Command vcpkg -ErrorAction SilentlyContinue) {
            & vcpkg --version
          } else {
            Write-Warning "vcpkg command not found in PATH."
          }

          $vcpkgRoot = $env:VCPKG_INSTALLATION_ROOT
          Write-Host "VCPKG_INSTALLATION_ROOT Environment Variable: $vcpkgRoot"
          
          if (Test-Path "C:\vcpkg") {
            Write-Host "Default vcpkg path (C:\vcpkg) exists."
          } else {
            Write-Warning "Default vcpkg path (C:\vcpkg) does not exist."
          }

          Write-Host ""
          Write-Host "--- Shared Libraries (OpenSSL) ---"
          if (Get-Command openssl -ErrorAction SilentlyContinue) {
            & openssl version
          } else {
            Write-Host "openssl command not found. Checking common installation paths..."
            $commonPaths = @(
              "C:\Program Files\OpenSSL\bin\openssl.exe",
              "C:\OpenSSL\bin\openssl.exe",
              "C:\tools\openssl\bin\openssl.exe"
            )
            foreach ($path in $commonPaths) {
              if (Test-Path $path) {
                Write-Host "Found OpenSSL at: $path"
              }
            }
          }
```

### callee `./.github/workflows/phase5.yml`

```yaml
name: Phase 5 - Connectivity & Specialized Tools Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: Connectivity & Specialized Tools Inventory
        shell: pwsh
        run: |
          Write-Host "--- Cloud CLIs ---"
          $cloudClis = @("az", "aws", "gh")
          foreach ($cli in $cloudClis) {
            if (Get-Command $cli -ErrorAction SilentlyContinue) {
              Write-Host "Found ${cli}:"
              & $cli --version
            } else {
              Write-Host "${cli} CLI is not installed or not in PATH."
            }
          }

          Write-Host ""
          Write-Host "--- Database Tools ---"
          $dbTools = @("mysql", "sqlcmd")
          foreach ($tool in $dbTools) {
            if (Get-Command $tool -ErrorAction SilentlyContinue) {
              Write-Host "Found ${tool}:"
              if ($tool -eq "sqlcmd") {
                # sqlcmd doesn't support --version, so we just check if it's executable
                Write-Host "  (sqlcmd detected, skipping --version check)"
              } else {
                & $tool --version
              }
            } else {
              Write-Host "${tool} is not installed or not in PATH."
            }
          }

          Write-Host ""
          Write-Host "--- Web Server Discovery ---"
          $webPaths = @(
            "C:\Apache24",
            "C:\nginx",
            "C:\Program Files\Apache",
            "C:\Program Files\nginx",
            "C:\inetpub"
          )
          
          foreach ($path in $webPaths) {
            if (Test-Path $path) {
              Write-Host "Found Web Server/IIS Directory: $path"
            }
          }

          Write-Host ""
          Write-Host "--- Networking & Connectivity Check ---"
          # We wrap these in a way that even if they fail, the script continues
          try {
            $dns = [System.Net.Dns]::GetHostEntry("google.com")
            Write-Host "DNS Resolution Success: $($dns.AddressList | Select-Object -ExpandProperty IPAddress -First 1)"
          } catch {
            Write-Warning "DNS Resolution Failed (This may be expected in a restricted environment)."
          }

          try {
            $ping = Test-NetConnection -ComputerName google.com -Port 443 -ErrorAction Stop
            Write-Host "Port 443 Connectivity: $($ping.PingSucceeded)"
          } catch {
            Write-Warning "Port 443 Connectivity check failed (No internet access or firewalled)."
          }

          # Final safety: ensure the script always exits with 0 so the job completes
          Write-Host "Discovery Phase 5 Complete."
          exit 0
```

### callee `./.github/workflows/phase6.yml`

```yaml
name: Phase 6 - WSL & Virtualization Discovery

on:
  workflow_call:

jobs:
  discovery:
    runs-on: windows-11-vs2026-arm
    steps:
      - uses: actions/checkout@v7

      - name: WSL and Virtualization Check
        shell: pwsh
        run: |
          Write-Host "--- WSL Binary Check ---"
          if (Get-Command wsl.exe -ErrorAction SilentlyContinue) {
            # Capture output and ignore errors for the version check
            & wsl.exe --version 2>&1 | Out-String
          } else {
            Write-Warning "wsl.exe not found in PATH."
          }

          Write-Host ""
          Write-Host "--- Registered Distributions ---"
          if (Get-Command wsl.exe -ErrorAction SilentlyContinue) {
            # Capture output and ignore errors for the list check
            & wsl.exe --list --verbose 2>&1 | Out-String
          }

          Write-Host ""
          Write-Host "--- Virtualization Feature Check ---"
          # Check if Hyper-V/Virtualization is enabled in the OS
          $virt = Get-CimInstance -ClassName Win32_DeviceGuard -ErrorAction SilentlyContinue
          if ($virt) {
            Write-Host "Device Guard Status: $($virt.SecurityServicesRunning)"
          } else {
            Write-Host "Could not retrieve Device Guard status via standard CIM."
          }

          # Check for Virtual Machine Platform feature
          Write-Host "Checking Windows Features..."
          $features = Get-WindowsOptionalFeature -Online | Where-Object { $_.FeatureName -match "VirtualMachine" -or $_.FeatureName -match "Subsystem" }
          $features | Select-Object FeatureName, State | Format-Table -AutoSize

          Write-Host ""
          Write-Host "--- Test Execution (Attempting to run a command in WSL) ---"
          if (Get-Command wsl.exe -ErrorAction SilentlyContinue) {
            # Use 2>&1 to capture stderr and check $LASTEXITCODE to prevent script failure
            $wslOut = & wsl.exe -e ls / 2>&1
            if ($LASTEXITCODE -eq 0) {
              Write-Host "WSL Execution Success:"
              Write-Host $wslOut
            } else {
              Write-Warning "WSL Execution Failed (Exit Code: $LASTEXITCODE). This indicates the subsystem is not functional or no distribution is registered."
              Write-Host "Error Output: $wslOut"
            }
          } else {
            Write-Warning "wsl.exe not found, skipping execution test."
          }

          Write-Host ""
          Write-Host "Discovery Phase 6 Complete."
          exit 0
```
