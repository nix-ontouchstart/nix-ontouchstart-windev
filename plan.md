> The goal is to "learn" windows-11-vs2026-arm in GitHub CI environment in iterative batch mode. Make a plan.

==== thinking

The user wants to "learn" the `windows-11-vs2026-arm` GitHub Actions runner environment iteratively.
I have access to:
1.  `dev.yml` which uses `windows-11-vs2026-arm`.
2.  A readme for `Windows11-VS2026-Arm64-Readme.md` which lists installed software, SDKs, .NET versions, etc.

To "learn" an environment in a CI context usually means:
1.  Identifying available tools and their versions.
2.  Testing basic commands (bash, powershell, git, etc.).
3.  Verifying specific developer tools (Visual Studio, MSBuild, dotnet, vcpkg).
4.  Understanding the environment variables.
5.  Checking networking/proxy settings (if any).
6.  Verifying permissions and filesystem structure.

Since I am an AI assistant, "learning" means I should create a series of tests/commands to run in the CI to gather information about the environment and then analyze those results.

Plan:
1.  **Baseline Environment Info**: Gather basic OS, architecture, and environment variables.
2.  **Tool Inventory Verification**: Check versions of key tools listed in the Readme (Python, Node, Go, Rust, etc.) to ensure they are accessible and correct.
3.  **Visual Studio & Build Tools**: Verify `vsdevelop`, `msbuild`, `dotnet`, and `vcpkg` setup. This is crucial for the `windev` job.
4.  **Network & Connectivity**: Check basic reachability.
5.  **FileSystem & Permissions**: Check common paths and write permissions.
6.  **Integration Tests**: Run a small sample build or compilation (if possible/appropriate) to see if the toolchain is correctly linked.

I should structure this as a series of "Discovery" steps.

Iterative Batch Mode:
- Batch 1: Basic Environment & Shells.
- Batch 2: Language Runtimes & Package Managers.
- Batch 3: .NET & Visual Studio Toolchain.
- Batch 4: Native Build Tools (C++, vcpkg, CMake).
- Batch 5: Specialized tools (Docker, Azure, etc. if needed).

Wait, the user wants me to *make a plan*. I should provide a structured plan.

==== thinkingSignature

==== text

To "learn" the `windows-11-vs2026-arm` environment effectively, we need to systematically verify the capabilities, tool versions, and environment configurations provided in the documentation against the actual runtime behavior in the GitHub Actions runner.

Here is the iterative plan to explore and validate this environment:

### Phase 1: Foundation & Shell Verification
**Goal:** Confirm basic OS info, architecture, and shell availability.
1.  **OS & Architecture:** Verify `uname -m`, `systeminfo`, and environment variables like `PROCESSOR_ARCHITECTURE`.
2.  **Shells:** Test `bash`, `powershell`, and `pwsh` (PowerShell 7) availability and basic command execution.
3.  **Environment Variables:** Dump all environment variables to identify custom paths, secrets (masked), and proxy settings.

### Phase 2: Language Runtimes & Package Managers
**Goal:** Verify the "Language and Runtime" and "Package Management" sections of the Readme.
1.  **Runtime Check:** Run `--version` for Python, Node.js, Go, Ruby, PHP, and Julia.
2.  **Package Managers:** Verify `npm`, `pip`, `composer`, `nuget`, `yarn`, and `vcpkg`.
3.  **Rust Toolchain:** Verify `rustc`, `cargo`, and `rustup` versions and the listed packages (bindgen, clippy, etc.).

### Phase 3: .NET & Visual Studio Toolchain (Critical for Windev)
**Goal:** Deep dive into the build environment for the `windev` job.
1.  **.NET SDKs:** List all installed .NET SDKs and Runtimes (`dotnet --list-sdks`, `dotnet --list-runtimes`).
2.  **MSBuild/Visual Studio:** Verify `msbuild` path, `vsdevelop` availability, and Visual Studio instance information.
3.  **Workloads:** Check if the specific workloads (e.g., `maui.windows`, `netcore.sdk`) are correctly registered in the VS instance.
4.  **Windows SDKs:** Confirm the presence of the 10.0.26100.0 and 10.0.22621.0 SDKs.

### Phase 4: Native Build & C++ Tools
**Goal:** Verify the C++ toolchain and native build capabilities.
1.  **Compilers:** Verify `gcc`, `clang` (via LLVM), and MSVC (`cl.exe`) paths.
2.  **Build Systems:** Verify `cmake`, `ninja`, `bazel`, and `make`.
3.  **vcpkg:** Test `vcpkg` command execution and check the `VCPKG_INSTALLATION_ROOT`.
4.  **Libraries:** Verify `OpenSSL` and other shared libraries.

### Phase 5: Connectivity & Specialized Tools
**Goal:** Verify external integrations and specialized CLI tools.
1.  **Cloud CLIs:** Test `az`, `aws`, `gh` (GitHub CLI) availability.
2.  **Web Servers:** Verify if Apache/Nginx are reachable (even if stopped) and check config paths.
3.  **Database Tools:** Verify `mysql` and `sqlcmd` (or equivalent) availability.

### Execution Strategy
I will implement this by creating a "discovery" script (e.g., `discovery.sh` or `discovery.ps1`) that I will run in the `dev.yml` workflow or a dedicated debug workflow. I will analyze the output of each phase before proceeding to the next.

> The scripts should be written as powershell commands in Github Action jobs. Follow the pattern bellow

caller `./.github/workflows/dev.yml`

```yaml
on: push
jobs:
  phase1:
    uses: ./.github/workflows/phase1.yml

  windev:
    needs: phase1
    runs-on: windows-11-vs2026-arm
    steps:
    - uses: actions/checkout@v7
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
        shell: powershell
        run: |
          Write-Host "--- Architecture ---"
          Write-Host ([System.Environment]::ProcessorArchitecture)
          Write-Host "--- System Info ---"
          Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, OSArchitecture | Format-List
          Write-Host "--- Environment Variables ---"
          Get-ChildItem Env: | Sort-Object Name | Select-Object Name, Value | Out-String

      - name: Shell Versions
        shell: powershell
        run: |
          Write-Host "--- Bash ---"
          bash --version
          Write-Host "--- PowerShell ---"
          $PSVersionTable.PSVersion
          Write-Host "--- pwsh ---"
          pwsh --version

```


