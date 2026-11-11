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


**Phase 1: Foundation & Shell Verification** (Completed/Defined)
**Phase 2: PowerShell & Rust Toolchain**
*   **PowerShell:** Verify version, list pre-installed modules (Az, AWSPowershell, Pester, etc.), and check for specific environment configurations.
*   **Rust:** Verify `rustc`, `cargo`, and `rustup` versions. Check for installed components like `bindgen` and `cargo-audit`.
**Phase 3: .NET & Visual Studio Toolchain**
*   Verify .NET SDKs, MSBuild paths, and VS Workloads.
**Phase 4: Native Build & C++ Tools**
*   Verify `cmake`, `ninja`, `vcpkg`, and LLVM/Clang paths.
**Phase 5: Connectivity & Specialized Tools**
*   Verify Cloud CLIs and database tool availability.

### Execution Strategy
I will implement this by creating a "discovery" script (e.g., `discovery.sh` or `discovery.ps1`) that I will run in the `dev.yml` workflow or a dedicated debug workflow. I will analyze the output of each phase before proceeding to the next.

> The scripts should be written as powershell commands in Github Action jobs. Follow the pattern bellow

caller `./.github/workflows/dev.yml`

```yaml
on: push
jobs:
  phase1:
    uses: ./.github/workflows/phase1.yml

  phase2:
    needs: phase1
    uses: ./.github/workflows/phase2.yml
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

callee `./.github/workflows/phase1.yml`

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
