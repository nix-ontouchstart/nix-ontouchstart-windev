# Learning Plan: windows-11-vs2026-arm

The goal is to "learn" the `windows-11-vs2026-arm` image in GitHub CI iteratively.

## Status Overview
- [x] Phase 1: Foundation & Shell Verification (Completed)
- [x] Phase 2: PowerShell & Rust Toolchain (Completed)
- [x] Phase 3: .NET & Visual Studio Toolchain (Completed)
- [ ] Phase 4: Native Build & C++ Tools (Next)
- [ ] Phase 5: Connectivity & Specialized Tools (Final)

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

### Phase 4: Native Build & C++ Tools (NEXT)
**Goal:** Verify the C++ toolchain and native build capabilities.
1. **Compilers:** Verify `gcc`, `clang` (via LLVM), and MSVC (`cl.exe`) paths.
2. **Build Systems:** Verify `cmake`, `ninja`, `bazel`, and `make`.
3. **vcpkg:** Test `vcpkg` command execution and check the `VCPKG_INSTALLATION_ROOT` (C:\vcpkg).
4. **Libraries:** Verify `OpenSSL` and other shared libraries.

### Phase 5: Connectivity & Specialized Tools
**Goal:** Verify external integrations and specialized CLI tools.
1. **Cloud CLIs:** Test `az`, `aws`, `gh` (GitHub CLI) availability.
2. **Database Tools:** Verify `mysql` and `sqlcmd` availability.
3. **Web Servers:** Verify Apache/Nginx config paths and status.

The scripts are written as powershell commands in Github Action jobs. Follow the pattern bellow

caller `./.github/workflows/dev.yml`

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

```

callee `./.github/workflows/phase1.yml`

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

callee `./.github/workflows/phase2.yml`

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

callee `./.github/workflows/phase3.yml`

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
