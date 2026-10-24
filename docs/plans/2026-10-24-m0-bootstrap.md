# M0 Bootstrap Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A public, source-only repository with the layered .NET 10 solution skeleton,
a 64-bit .NET Framework 4.8 Bridge stub, CI that enforces the style, architecture,
clean-room and source-only rules, and the baseline docs, delivered as four batch PRs.

**Architecture:** Eight `src/` projects follow charter 6.2 and 6.3. A WPF Generic Host
composition root opens an empty shell. The Bridge is a separate net48 x64 exe that
prints its identity. Architecture tests read `.csproj` files, assemblies (NetArchTest)
and source syntax (Roslyn) to enforce the rules.

**Tech Stack:** .NET SDK 10.0.401, WPF, Microsoft.Extensions.Hosting, CommunityToolkit.Mvvm,
.NET Framework 4.8, xUnit v3 on Microsoft Testing Platform, Shouldly, NetArchTest, Roslyn,
GitHub Actions, gitleaks.

**Spec:** `docs/specs/2026-10-24-m0-bootstrap-design.md` (and the charter, `docs/PROJECT.md`
once Task 1 adds it).

## Global Constraints

- SDK `10.0.401`, `rollForward: latestFeature`; test runner `Microsoft.Testing.Platform` in `global.json`.
- Targets: Domain, Application, Plugins.Abstractions `net10.0`; Infrastructure, Presentation, Updater `net10.0-windows`; Bridge `net48`; Bridge.Contracts `netstandard2.0`.
- Bridge `PlatformTarget` = `$(BridgePlatform)`, default `x64`; `-p:BridgePlatform=x86` builds 32-bit.
- Single version `0.1.0` in `Directory.Build.props`; `Authors` "Jamie Head", `Company` "NyxionSoftware", `Product` "EMC Companion", `Copyright` "Copyright (c) 2026 NyxionSoftware".
- `Nullable` enable, `ImplicitUsings` disable, `LangVersion` latest, `TreatWarningsAsErrors` in Release, `EnforceCodeStyleInBuild` true, central package management, package lock files.
- Style (charter 7): block-scoped namespaces, usings outside the namespace with System first and groups separated by a blank line, Allman braces, braces always, explicit access modifiers, `_camelCase` private readonly fields, one public type per file named like the file, no `#region`, `sealed` unless designed for inheritance.
- Test classes `<Type>Tests`, test methods `Method_State_Expected`, Shouldly assertions.
- Package versions: Microsoft.Extensions.Hosting 10.0.13, Microsoft.Extensions.DependencyInjection.Abstractions 10.0.13, CommunityToolkit.Mvvm 8.4.2, xunit.v3 4.0.2, Shouldly 4.3.0, NetArchTest.eNhancedEdition 1.4.5, Microsoft.CodeAnalysis.CSharp 5.9.0, Microsoft.NETFramework.ReferenceAssemblies 1.0.3; gitleaks v8.30.1.
- Source only: never commit `*.dll`, `*.exe`, `*.pdb`, `*.nupkg`, `*.snupkg`, `*.zip`, `*.msi`, `*.7z`, or anything under `bin/`, `obj/`, `artifacts/`, `publish/`.
- Clean room: no Oracle files, no decompiled code, no metadata tokens, no workstation hostnames, no `C:\Users\<name>` paths, no real customer names. Placeholders use `example.invalid` and `not-a-password`.
- Never open a visible app window during automated work; tests never create windows.
- Commits use the repo-local identity `Jamie Head <141502076+NyxionSoftware@users.noreply.github.com>` and end with `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`.
- Every `gh` command that changes GitHub state is shown to the owner and approved before it runs.
- One PR per batch into `main-dev`; the next batch branches from `main-dev` only after the owner says "Approved" and the PR is squash-merged.

## Review Focus

1. **Paths with spaces.** The default deploy target (`C:\Program Files\Oracle Hospitality\EMC Companion\`) and many clone folders contain spaces; scripts and tests must quote and handle them (Task 10 runs the deploy script against a temp target with spaces).
2. **No `.git` folder.** Someone building from a GitHub source zip runs `dotnet test` without git; git-dependent checks must skip with a reason, not fail (Task 11 test).
3. **Legitimate binary assets.** PNG/ICO under `assets/` and PNG under `docs/` must pass the source-only rule; a `.dll` anywhere, including test fixture folders, must fail (Task 11 theory rows).
4. **Bridge safety config drift.** Someone editing `App.config` could drop `PerformEMCVersionCheck=0` or a binding redirect; a test must fail when they do (Task 8).
5. **Tests run from another working directory.** Visual Studio Test Explorer and CI start tests in different folders; repository-root discovery must walk up from the test assembly to `EmcCompanion.slnx` (Task 8 introduces `RepositoryRoot`, Tasks 11 and 12 reuse it).

---

## Task 0: Repository restructure and Trello board (planning session)

Done by the planning session with the owner approving each `gh` command; listed so the
record is complete. No code.

- [ ] Push `legacy` at `b166aeb` to the old repo; `gh repo rename EMC-Companion-Legacy`; set its default branch to `legacy`; delete its other branches.
- [ ] `gh repo create NyxionSoftware/EMC-Companion --public`; push local `main` and `main-dev` (root commit `478fa3b`); set default branch `main-dev`, squash and merge-commit only, delete head branches on merge.
- [ ] Enable secret scanning, push protection, Dependabot alerts and security updates, private vulnerability reporting.
- [ ] Rulesets phase 1 for `main` (merge commits only) and `main-dev` (squash only): no deletion, no force push, PR required with 0 approvals.
- [ ] Repoint old local clones' `origin` to `EMC-Companion-Legacy.git` so no legacy branch can be pushed to the new public repo.
- [ ] Create the private Trello board "EMC Companion Roadmap" from charter 16 (lists, labels, 65 batch cards, 14 milestone cards).

---

# Batch M0.1: Repository and legal

Branch `feature/m0-1-repo-legal` (exists; holds the spec and this plan). Trello card
"M0.1 Repository and legal" to In Progress.

### Task 1: Charter in the repo (`docs/PROJECT.md`)

**Files:**
- Create: `docs/PROJECT.md` (from the owner's charter `Project.md`)

**Interfaces:**
- Produces: `docs/PROJECT.md` section numbers stay identical to the charter, so the spec's references (for example "charter 6.3") resolve to `docs/PROJECT.md`.

- [ ] **Step 1: Copy the charter verbatim** from the owner's `Project.md` into `docs/PROJECT.md` and commit nothing yet.
- [ ] **Step 2: Apply the scrub rules (spec section 5).** Workstation hostname becomes "the owner's workstation"; `C:\Users\<name>\` becomes `%USERPROFILE%\`; REA tooling described generically.
- [ ] **Step 3: Apply the spec section 3 corrections in place** in charter 1.1 (EMC row), 3.4, 6.4 and 8.4, and set 1.1 "Repository" to "public `NyxionSoftware/EMC-Companion`; legacy private repo `EMC-Companion-Legacy`". Change `EmcCompanion.sln` to `EmcCompanion.slnx` in 6.1. In 17, fill the "Owner decision" column from spec section 2 and append rows D21-D25 with the spec's outcomes.
- [ ] **Step 4: Verify the scrub.**
  Run (on the owner's workstation, so the hostname comes from the environment and never appears in the repo): `Select-String -Path docs\PROJECT.md -Pattern $env:COMPUTERNAME,'C:\\Users\\[A-Za-z]'`
  Expected: no output.
- [ ] **Step 5: Commit** `docs/PROJECT.md` with message `Add the project charter as docs/PROJECT.md`.

### Task 2: Legal and policy files

**Files:**
- Create: `LICENSE`, `NOTICE`, `THIRD-PARTY-NOTICES.md`, `SECURITY.md`, `.github/CODEOWNERS`

- [ ] **Step 1: `LICENSE`** is the unmodified Apache License 2.0 text from `https://www.apache.org/licenses/LICENSE-2.0.txt`.
- [ ] **Step 2: `NOTICE`** is the charter 2.6 block exactly.
- [ ] **Step 3: `THIRD-PARTY-NOTICES.md`**: title, one sentence ("Open-source components shipped with EMC Companion and their licenses."), and a table with columns Component, Version, License, Source. No rows yet.
- [ ] **Step 4: `SECURITY.md`**: report through GitHub private vulnerability reporting (Security tab > Report a vulnerability); supported versions table (`main` latest release only); do not attach logs, exports or screenshots with real enterprise data (charter 2.5, 13).
- [ ] **Step 5: `.github/CODEOWNERS`** contains `* @NyxionSoftware`.
- [ ] **Step 6: Commit** the five files: `Add license, notice, security policy and code owners`.

### Task 3: README and CONTRIBUTING

**Files:**
- Modify: `README.md`
- Create: `CONTRIBUTING.md`

- [ ] **Step 1: `README.md`**: badges from the charter header (version 0.1.0-dev, .NET 10, Windows 10+ x64, license Apache-2.0, secret scan); one-paragraph description (charter intro, first two sentences); "Status: bootstrapping (M0)"; "Build from source" (install .NET SDK 10.0.401+, `git clone`, `dotnet build`, `dotnet test`; no Oracle files are needed to build); "Requires a licensed Oracle Simphony EMC client to do anything useful"; branch model (`main` stable, `main-dev` development); the not-affiliated notice from NOTICE; links to `docs/PROJECT.md`, `CONTRIBUTING.md`, `SECURITY.md`.
- [ ] **Step 2: `CONTRIBUTING.md`** sections: Workflow (charter 15.1 steps), Branches (charter 15.2 table), Code style (pointer to charter 7 and `.editorconfig`), Clean-room rule (charter 2.2 rules 1-7 verbatim), Sample data only (charter 2.5), Commit messages (charter 7.5), Secret scanning: install `pre-commit` (`pip install pre-commit`, then `pre-commit install`), and the fake-token self-test below.
- [ ] **Step 3: Fake-token self-test text** (exact copy for CONTRIBUTING.md):

  ````markdown
  To check that the hook works, generate a throwaway fake token, try to commit it,
  and confirm the commit is blocked. Never push it.

  ```powershell
  $fake = 'ghp_' + -join ((48..57) + (65..90) + (97..122) | Get-Random -Count 36 | ForEach-Object { [char]$_ })
  Set-Content -Path hook-test.txt -Value "token = $fake"
  git add hook-test.txt
  git commit -m "hook test"   # expected: blocked by gitleaks
  git reset hook-test.txt; Remove-Item hook-test.txt
  ```
  ````

- [ ] **Step 4: Commit**: `Add README build steps and contributing guide`.

### Task 4: Repository hygiene files

**Files:**
- Create: `.gitignore`, `.gitattributes`, `.editorconfig`, `.pre-commit-config.yaml`

- [ ] **Step 1: `.gitignore`**: Visual Studio essentials (`bin/`, `obj/`, `.vs/`, `*.user`, `*.suo`, `TestResults/`, `*.binlog`) plus charter 2.4 and 19.5 entries (`publish/`, `artifacts/`, `lib/emc/`, `**/EMCClient/`, `appsettings.Local.json`, `settings.json`, `*.pfx`, `logs/`, `screenshots/raw/`, `REA-Notes/`). Do not ignore `*.snk` (none are used).
- [ ] **Step 2: `.gitattributes`**: `* text=auto eol=crlf`; `*.png *.ico *.jpg *.gif *.ttf *.otf *.woff *.woff2 binary`.
- [ ] **Step 3: `.editorconfig`**: root; `[*]` utf-8, crlf, final newline, trim trailing whitespace, 4 spaces; `[*.{csproj,props,targets,slnx,json,xaml,yml,yaml,config,manifest}]` 2 spaces; `[*.md]` no trailing-whitespace trim. `[*.cs]` with these at `error`:
  - `csharp_style_namespace_declarations = block_scoped:error` and `dotnet_diagnostic.IDE0161.severity = error`
  - `csharp_using_directive_placement = outside_namespace:error`, `IDE0065` error
  - `dotnet_sort_system_directives_first = true`, `dotnet_separate_import_directive_groups = true`
  - `csharp_new_line_before_open_brace = all`, `csharp_prefer_braces = true:error`, `IDE0011` error
  - `IDE0005` (unnecessary usings), `IDE0040` (accessibility modifiers, `dotnet_style_require_accessibility_modifiers = always`), `IDE0044` (readonly) at error
  - `csharp_style_var_when_type_is_apparent = true:suggestion`, `csharp_style_var_elsewhere = false:suggestion`
  - Naming rules: private fields `_camelCase`; interfaces `I` prefix; async methods `Async` suffix; constants PascalCase; all at error.
- [ ] **Step 4: `.pre-commit-config.yaml`**: repo `https://github.com/gitleaks/gitleaks` at `rev: v8.30.1`, hook `gitleaks`. A comment notes ggshield (GitGuardian) as an optional second hook.
- [ ] **Step 5: Verify** `git status --short` lists only the four new files and `git check-ignore -v bin/x obj/x publish/x artifacts/x` reports all four as ignored.
- [ ] **Step 6: Commit**: `Add gitignore, gitattributes, editorconfig and pre-commit config`.

### Task 5: Ship M0.1

- [ ] **Step 1: Self-review** the branch diff against spec section 5 (every task item present, scrub clean).
- [ ] **Step 2: Push** `feature/m0-1-repo-legal` (git, no gh).
- [ ] **Step 3: Open the PR** with owner approval: `gh pr create --base main-dev --title "M0.1 Repository and legal" --body-file <body>`; body uses the charter 15.3 template with the Trello card link and the spec section 5 "What to test" list.
- [ ] **Step 4: Trello**: card to In Review, then Testing when the PR is ready. Wait for the owner's "Approved" and squash-merge.

---

# Batch M0.2: Solution skeleton

Branch `feature/m0-2-solution-skeleton` from the updated `main-dev`. Trello card to In Progress.

### Task 6: Build infrastructure files

**Files:**
- Create: `global.json`, `nuget.config`, `Directory.Build.props`, `Directory.Packages.props`, `EmcCompanion.slnx`

- [ ] **Step 1: `global.json`**: `sdk.version` `10.0.401`, `rollForward` `latestFeature`, `allowPrerelease` false; `test.runner` `Microsoft.Testing.Platform`.
- [ ] **Step 2: `nuget.config`**: `<clear />` then `nuget.org` (`https://api.nuget.org/v3/index.json`).
- [ ] **Step 3: `Directory.Build.props`**: Global Constraints values plus `Deterministic` true, `RestorePackagesWithLockFile` true, `AnalysisLevel` `latest-recommended`, `GenerateDocumentationFile` true everywhere (IDE0005 only fires in build when it is on), `NoWarn` `$(NoWarn);CS1591` except for `EmcCompanion.Plugins.Abstractions` and `EmcCompanion.Bridge.Contracts` (condition on `$(MSBuildProjectName)`), `PackageProjectUrl` `https://nyxionsoftware.com`, `RepositoryUrl` `https://github.com/NyxionSoftware/EMC-Companion`.
- [ ] **Step 4: `Directory.Packages.props`**: `ManagePackageVersionsCentrally` true and a `PackageVersion` for every package in Global Constraints.
- [ ] **Step 5: `EmcCompanion.slnx`**: `dotnet new sln --name EmcCompanion` (creates `.slnx` on SDK 10); solution folders `src` and `tests` get filled as projects are added.
- [ ] **Step 6: `tests/Directory.Build.props`**: imports the parent props (`$([MSBuild]::GetPathOfFileAbove('Directory.Build.props', '$(MSBuildThisFileDirectory)../'))`), sets `OutputType` Exe (xUnit v3 test projects are executables) and `IsPackable` false, and adds `PackageReference`s for xunit.v3 and Shouldly so test projects list only their project references.
- [ ] **Step 7: Verify** `dotnet --version` prints `10.0.4xx`.
- [ ] **Step 8: Commit**: `Add SDK pin, NuGet source, shared build props and solution`.

If an analyzer rule from `latest-recommended` conflicts with charter 7 (for example a
rule that wants file-scoped namespaces or internal types in the exe), set that rule's
severity in `.editorconfig` with a one-line comment giving the charter reason. Never
weaken a charter rule to satisfy an analyzer.

### Task 7: Library projects and the contract version

**Files:**
- Create: `src/EmcCompanion.Domain/EmcCompanion.Domain.csproj`, `src/EmcCompanion.Domain/DomainAssemblyMarker.cs`
- Create: `src/EmcCompanion.Application/EmcCompanion.Application.csproj`, `src/EmcCompanion.Application/ApplicationServiceCollectionExtensions.cs`
- Create: `src/EmcCompanion.Infrastructure/EmcCompanion.Infrastructure.csproj`, `src/EmcCompanion.Infrastructure/InfrastructureServiceCollectionExtensions.cs`
- Create: `src/EmcCompanion.Updater/EmcCompanion.Updater.csproj`, `src/EmcCompanion.Updater/UpdaterServiceCollectionExtensions.cs`
- Create: `src/EmcCompanion.Bridge.Contracts/EmcCompanion.Bridge.Contracts.csproj`, `src/EmcCompanion.Bridge.Contracts/ContractInfo.cs`
- Create: `src/EmcCompanion.Plugins.Abstractions/EmcCompanion.Plugins.Abstractions.csproj`, `src/EmcCompanion.Plugins.Abstractions/PluginApi.cs`
- Test: `tests/EmcCompanion.Bridge.Contracts.Tests/EmcCompanion.Bridge.Contracts.Tests.csproj`, `tests/EmcCompanion.Bridge.Contracts.Tests/ContractInfoTests.cs`

**Interfaces:**
- Produces:
  - `namespace EmcCompanion.Domain { public static class DomainAssemblyMarker { } }`
  - `namespace EmcCompanion.Application { public static class ApplicationServiceCollectionExtensions { public static IServiceCollection AddApplication(this IServiceCollection services); } }`
  - `EmcCompanion.Infrastructure.InfrastructureServiceCollectionExtensions.AddInfrastructure(this IServiceCollection services) : IServiceCollection`
  - `EmcCompanion.Updater.UpdaterServiceCollectionExtensions.AddUpdater(this IServiceCollection services) : IServiceCollection`
  - `namespace EmcCompanion.Bridge.Contracts { public static class ContractInfo { public const int Version = 1; } }`
  - `namespace EmcCompanion.Plugins { public static class PluginApi { public const string Version = "1.0"; } }` (assembly `EmcCompanion.Plugins.Abstractions`, `RootNamespace` `EmcCompanion.Plugins`, matching charter 11.3)
  - Each `Add*` returns `services` unchanged in M0, with an XML summary saying what it will register.
- Project references (charter 6.3): Application → Domain, Plugins.Abstractions; Infrastructure → Application, Domain, Bridge.Contracts, Plugins.Abstractions; Updater → Application. Application, Infrastructure and Updater reference `Microsoft.Extensions.DependencyInjection.Abstractions`.

- [ ] **Step 1: Write the failing test** `ContractInfoTests.Version_Initial_IsOne`: `ContractInfo.Version.ShouldBe(1);` in `namespace EmcCompanion.Bridge.Contracts.Tests`. Test project targets `net10.0` and references Bridge.Contracts (packages come from `tests/Directory.Build.props`).
- [ ] **Step 2: Run** `dotnet test --project tests/EmcCompanion.Bridge.Contracts.Tests`. Expected: build FAILS, `ContractInfo` not found.
- [ ] **Step 3: Create the six library projects** with the targets in Global Constraints and the types in the Interfaces block; XML doc comments on every public member of Bridge.Contracts and Plugins.Abstractions. Add all projects to the slnx under `src`/`tests`.
- [ ] **Step 4: Run** `dotnet test --project tests/EmcCompanion.Bridge.Contracts.Tests`. Expected: 1 passed.
- [ ] **Step 5: Run** `dotnet build -c Release`. Expected: 0 warnings, 0 errors (proves CS1591 is satisfied in the two public-API projects and style rules hold).
- [ ] **Step 6: Commit** projects, test and `packages.lock.json` files: `Add the library projects and the contract version`.

### Task 8: Bridge stub and its safety configuration

**Files:**
- Create: `src/EmcCompanion.Bridge/EmcCompanion.Bridge.csproj`, `src/EmcCompanion.Bridge/Program.cs`, `src/EmcCompanion.Bridge/App.config`
- Test: `tests/EmcCompanion.Architecture.Tests/EmcCompanion.Architecture.Tests.csproj`, `tests/EmcCompanion.Architecture.Tests/RepositoryRoot.cs`, `tests/EmcCompanion.Architecture.Tests/BridgeConfigurationTests.cs`

**Interfaces:**
- Consumes: `ContractInfo.Version` (Task 7).
- Produces:
  - `internal static class RepositoryRoot { public static DirectoryInfo Find(); }` in `namespace EmcCompanion.Architecture.Tests`: walks up from `AppContext.BaseDirectory` to the first folder containing `EmcCompanion.slnx`; throws `InvalidOperationException("EmcCompanion.slnx not found above <dir>")` if none.
  - Bridge console output, exactly three lines:
    ```
    EMC Companion Bridge <major.minor.patch>
    Contract version <ContractInfo.Version>
    Process <64-bit|32-bit>
    ```
    Bitness comes from `Environment.Is64BitProcess`, never the OS.

- [ ] **Step 1: Write the failing tests** in `BridgeConfigurationTests` (`namespace EmcCompanion.Architecture.Tests`), each loading `src/EmcCompanion.Bridge/App.config` via `RepositoryRoot.Find()` with `XDocument`:
  - `AppSettings_PerformEmcVersionCheck_IsZero`: the `appSettings/add[@key='PerformEMCVersionCheck']` value is `"0"`.
  - `AppSettings_EGatewayTimeout_MatchesEmcDefault`: value `"1000"`.
  - `Runtime_LegacyCorruptedStateExceptionsPolicy_IsEnabled`: `runtime/legacyCorruptedStateExceptionsPolicy/@enabled` is `"true"`.
  - `Runtime_LegacyPathHandling_IsEnabled`: `runtime/AppContextSwitchOverrides/@value` contains `Switch.System.IO.UseLegacyPathHandling=true`.
  - `BindingRedirects_EmcDependencies_MatchEmcExpectations` (theory): each row asserts a `dependentAssembly` with that name and public key token redirects `0.0.0.0-<v>` to `<v>`:

    | name | publicKeyToken | version |
    |---|---|---|
    | Newtonsoft.Json | 30ad4fe6b2a6aeed | 13.0.0.0 |
    | System.Memory | cc7b13ffcd2ddd51 | 4.0.2.0 |
    | System.Buffers | cc7b13ffcd2ddd51 | 4.0.4.0 |
    | System.Runtime.CompilerServices.Unsafe | b03f5f7f11d50a3a | 6.0.1.0 |
    | System.Text.Json | cc7b13ffcd2ddd51 | 8.0.0.5 |
    | System.Formats.Asn1 | cc7b13ffcd2ddd51 | 9.0.0.15 |
    | System.Numerics.Vectors | b03f5f7f11d50a3a | 4.1.5.0 |

  - `SupportedRuntime_Framework_Is48`: `startup/supportedRuntime/@sku` is `.NETFramework,Version=v4.8`.
  - `RepositoryRootTests.Find_FromAnotherWorkingDirectory_ReturnsFolderWithSolution` (Review Focus 5): sets `Environment.CurrentDirectory` to `Path.GetTempPath()`, calls `RepositoryRoot.Find()`, restores the directory in `finally`; the result contains `EmcCompanion.slnx`.
  Test project: `net10.0-windows`, `UseWPF` true (later tasks load Presentation).
- [ ] **Step 2: Run** `dotnet test --project tests/EmcCompanion.Architecture.Tests`. Expected: FAIL, `App.config` not found.
- [ ] **Step 3: Create the Bridge project**: `net48`, `OutputType` Exe, `AssemblyName` `EmcCompanion.Bridge`, `PlatformTarget` `$(BridgePlatform)` with `<BridgePlatform Condition="'$(BridgePlatform)' == ''">x64</BridgePlatform>`, `Prefer32Bit` false, `Microsoft.NETFramework.ReferenceAssemblies` with `PrivateAssets` all, references Bridge.Contracts only. `App.config` holds exactly the values the tests assert. `Program.Main` writes the three lines and returns 0. Add to slnx.
- [ ] **Step 4: Run** `dotnet test --project tests/EmcCompanion.Architecture.Tests`. Expected: all passed.
- [ ] **Step 5: Verify bitness** (the CI step in Task 13 repeats the 64-bit check):
  Run: `dotnet build src/EmcCompanion.Bridge -c Release; & src\EmcCompanion.Bridge\bin\Release\net48\EmcCompanion.Bridge.exe`
  Expected: `EMC Companion Bridge 0.1.0`, `Contract version 1`, `Process 64-bit`.
  Run: `dotnet build src/EmcCompanion.Bridge -c Release -p:BridgePlatform=x86; & src\EmcCompanion.Bridge\bin\Release\net48\EmcCompanion.Bridge.exe`
  Expected: last line `Process 32-bit`. Rebuild without the switch afterwards.
- [ ] **Step 6: Commit**: `Add the 64-bit Bridge stub with EMC's runtime settings and tests`.

### Task 9: WPF shell and composition root

**Files:**
- Create: `src/EmcCompanion.Presentation/EmcCompanion.Presentation.csproj`, `App.xaml`, `App.xaml.cs`, `Hosting/AppHost.cs`, `PresentationServiceCollectionExtensions.cs`, `Shell/ShellViewModel.cs`, `Shell/ShellWindow.xaml`, `Shell/ShellWindow.xaml.cs`
- Test: `tests/EmcCompanion.Presentation.Tests/EmcCompanion.Presentation.Tests.csproj`, `AppHostTests.cs`, `ShellViewModelTests.cs`

**Interfaces:**
- Consumes: `AddApplication`, `AddInfrastructure`, `AddUpdater` (Task 7).
- Produces:
  - `namespace EmcCompanion.Presentation.Hosting { public static class AppHost { public static IHost Build(string[] args); } }`: `Host.CreateApplicationBuilder(args)`, calls `AddApplication()`, `AddInfrastructure()`, `AddUpdater()`, `AddPresentation()`, logging cleared then `AddDebug()`, service provider options `ValidateOnBuild = true`, `ValidateScopes = true`. Only this namespace references Infrastructure.
  - `EmcCompanion.Presentation.PresentationServiceCollectionExtensions.AddPresentation(this IServiceCollection services) : IServiceCollection`: registers `ShellViewModel` and `ShellWindow` as transient.
  - `namespace EmcCompanion.Presentation.Shell { public sealed class ShellViewModel : ObservableObject { public string Title { get; } } }`: `Title` is `"EMC Companion"`.
  - `ShellWindow(ShellViewModel viewModel)` sets `DataContext`; `Title="{Binding Title}"`, 1280x800, centered, empty content.
  - `App`: no `StartupUri`; `OnStartup` builds the host with `AppHost.Build(e.Args)`, `StartAsync`, resolves `ShellWindow`, sets `MainWindow`, `Show()`. `OnExit` awaits `StopAsync` and disposes the host. `ShutdownMode` `OnMainWindowClose`.
  - Project: `net10.0-windows`, `UseWPF` true, `OutputType` WinExe, `AssemblyName` `EmcCompanion`, `RootNamespace` `EmcCompanion.Presentation`; references Application, Infrastructure, Updater; packages Microsoft.Extensions.Hosting, CommunityToolkit.Mvvm.

- [ ] **Step 1: Write the failing tests** (`namespace EmcCompanion.Presentation.Tests`, project `net10.0-windows`, `UseWPF` true):
  - `AppHostTests.Build_WithNoArguments_ValidatesAllRegistrations`: `using var host = AppHost.Build(Array.Empty<string>());` completes without throwing.
  - `ShellViewModelTests.Title_ByDefault_IsEmcCompanion`: resolve `ShellViewModel` from `AppHost.Build(...)`'s services; `Title.ShouldBe("EMC Companion")`. Neither test resolves `ShellWindow`.
- [ ] **Step 2: Run** `dotnet test --project tests/EmcCompanion.Presentation.Tests`. Expected: build FAILS, `AppHost` not found.
- [ ] **Step 3: Implement** the Interfaces block. Add to slnx.
- [ ] **Step 4: Run** `dotnet test`. Expected: all tests in the solution pass.
- [ ] **Step 5: Owner check (not automated, charter 3.3):** list in the PR's What to test that the owner runs `EmcCompanion.exe` and sees the empty "EMC Companion" window open and close.
- [ ] **Step 6: Add rows** for Microsoft.Extensions.Hosting (MIT) and CommunityToolkit.Mvvm (MIT) to `THIRD-PARTY-NOTICES.md`.
- [ ] **Step 7: Commit**: `Add the WPF shell with a Generic Host composition root`.

### Task 10: Local deploy script

**Files:**
- Create: `build/Deploy-Local.ps1`

**Interfaces:**
- Produces: `Deploy-Local.ps1 [-Target <path>] [-Configuration Release]` with `[CmdletBinding(SupportsShouldProcess)]`. Default `-Target` `C:\Program Files\Oracle Hospitality\EMC Companion`.

- [ ] **Step 1: Write the script**: resolves the repo root from `$PSScriptRoot`; `dotnet publish src/EmcCompanion.Presentation -c $Configuration -r win-x64 --self-contained true -o publish/local/app`; `dotnet publish src/EmcCompanion.Bridge -c $Configuration -o publish/local/app/bridge`; if the target is not writable by the current process, stops with "Run this from an elevated PowerShell to deploy to <target>." (so elevation is required only where it is needed, and CI can deploy to a temp folder); if an `EmcCompanion` process runs from the target, asks before stopping it; copies everything except an existing `appsettings.json`; prints `Deployed EMC Companion <version> to <target>`. All paths quoted.
- [ ] **Step 2: Verify with spaces** (Review Focus 1):
  Run: `$t = Join-Path $env:TEMP 'EMC Companion Deploy Test'; .\build\Deploy-Local.ps1 -Target $t`
  Expected: `Deployed EMC Companion 0.1.0 to ...EMC Companion Deploy Test`; `$t\EmcCompanion.exe` and `$t\bridge\EmcCompanion.Bridge.exe` exist. Run `.\build\Deploy-Local.ps1 -Target $t -WhatIf` and confirm nothing is copied. Delete `$t`.
- [ ] **Step 3: Verify the guard**: from a non-elevated PowerShell, `.\build\Deploy-Local.ps1` stops with the elevation message and copies nothing.
- [ ] **Step 4: Commit**: `Add the local deploy script`.

### Ship M0.2

- [ ] Self-review against spec section 6, push, open the PR with owner approval (`gh pr create --base main-dev --title "M0.2 Solution skeleton"`), body with spec 6.5 "What to test", Trello card to In Review then Testing. Wait for "Approved" and the squash-merge.

---

# Batch M0.3: CI and architecture tests

Branch `feature/m0-3-ci` from the updated `main-dev`. Trello card to In Progress.

### Task 11: Source-only rule

**Files:**
- Create: `tests/EmcCompanion.Architecture.Tests/SourceOnlyRules.cs`, `tests/EmcCompanion.Architecture.Tests/SourceOnlyTests.cs`

**Interfaces:**
- Consumes: `RepositoryRoot.Find()` (Task 8).
- Produces: `internal static class SourceOnlyRules { public static IReadOnlyList<string> FindViolations(IEnumerable<string> trackedPaths); }`. Paths are repo-relative with `/` separators. A path is a violation if its extension is in the deny list (Global Constraints), or any segment is `bin`, `obj`, `artifacts` or `publish`, or it is an image (`.png .ico .jpg .jpeg .gif .bmp`) outside `assets/` and `docs/`, or a font (`.ttf .otf .woff .woff2`) outside `assets/`. Comparison is case-insensitive.

- [ ] **Step 1: Write the failing tests** in `SourceOnlyTests`:
  - `FindViolations_BuildOutputAndBinaries_AreReported` (theory, each row must be reported): `tests/Fixtures/test.dll`, `src/EmcCompanion.Bridge/bin/Release/net48/x.txt`, `src/A/obj/project.assets.json`, `artifacts/EmcCompanion-Setup-0.1.0.exe`, `publish/app/readme.txt`, `tools/x.PDB`, `samples/plugin.zip`, `src/EmcCompanion.Presentation/logo.png`, `docs/fonts/Inter.ttf`.
  - `FindViolations_SourceAndAllowedAssets_AreNotReported` (theory, none reported): `src/EmcCompanion.Domain/DomainAssemblyMarker.cs`, `assets/icons/app.ico`, `assets/fonts/Inter-Regular.ttf`, `docs/images/home.png`, `README.md`, `src/EmcCompanion.Presentation/packages.lock.json`.
  - `TrackedFiles_InRepository_HaveNoViolations`: runs `git ls-files -z` in `RepositoryRoot.Find()`; if git is not on PATH or the folder is not a work tree, calls `Assert.Skip("Not a git checkout; the source-only check runs in CI.")` (Review Focus 2); otherwise `FindViolations(...).ShouldBeEmpty()`.
- [ ] **Step 2: Run** `dotnet test --project tests/EmcCompanion.Architecture.Tests --filter-class EmcCompanion.Architecture.Tests.SourceOnlyTests`. Expected: build FAILS, `SourceOnlyRules` not found.
- [ ] **Step 3: Implement `SourceOnlyRules.FindViolations`.**
- [ ] **Step 4: Run** the same command. Expected: all passed.
- [ ] **Step 5: Verify the skip** (Review Focus 2): copy the repo without `.git` to a temp folder with a space in its name (`robocopy . "$env:TEMP\emc zip test" /E /XD .git bin obj`), run `dotnet test --project tests/EmcCompanion.Architecture.Tests` there. Expected: `TrackedFiles_InRepository_HaveNoViolations` reported as skipped with the message, the run succeeds. Delete the temp folder.
- [ ] **Step 6: Commit**: `Add the source-only rule as an architecture test`.

### Task 12: Architecture rules

**Files:**
- Create: `tests/EmcCompanion.Architecture.Tests/ProjectReferenceTests.cs`, `TypeDependencyTests.cs`, `SourceStyleTests.cs`
- Modify: `tests/EmcCompanion.Architecture.Tests/EmcCompanion.Architecture.Tests.csproj` (reference the seven `src` projects other than the Bridge; packages NetArchTest.eNhancedEdition, Microsoft.CodeAnalysis.CSharp)

**Interfaces:**
- Consumes: `RepositoryRoot.Find()`; marker and extension types from Tasks 7 and 9 to locate assemblies (`typeof(DomainAssemblyMarker).Assembly`, etc.).

- [ ] **Step 1: Write the failing tests:**
  - `ProjectReferenceTests.ProjectReferences_EachSourceProject_StayInsideCharterGraph` (theory over the eight `src` projects): parses `<ProjectReference Include>` from each `.csproj`; references must be a subset of: Domain {}, Application {Domain, Plugins.Abstractions}, Infrastructure {Application, Domain, Bridge.Contracts, Plugins.Abstractions}, Presentation {Application, Domain, Infrastructure, Plugins.Abstractions, Updater}, Updater {Application}, Bridge {Bridge.Contracts}, Bridge.Contracts {}, Plugins.Abstractions {}.
  - `ProjectReferenceTests.Projects_AnyProject_NeverReferenceEmcLibraries`: no `.csproj` under `src/` or `tests/` contains `lib/emc`, `lib\emc` or `EMCClient` (case-insensitive).
  - `TypeDependencyTests.Application_Types_DoNotDependOnOuterLayersOrWpf`: NetArchTest, no dependency on `EmcCompanion.Infrastructure`, `EmcCompanion.Presentation`, `EmcCompanion.Bridge.Contracts`, `System.Windows`.
  - `TypeDependencyTests.Domain_Types_DependOnNoOtherLayer`: no dependency on `EmcCompanion.Application`, `EmcCompanion.Infrastructure`, `EmcCompanion.Presentation`, `EmcCompanion.Updater`, `EmcCompanion.Bridge.Contracts`, `EmcCompanion.Plugins`, `System.Windows`.
  - `TypeDependencyTests.Presentation_InfrastructureUse_OnlyInHosting`: types in the Presentation assembly that depend on `EmcCompanion.Infrastructure` all reside in `EmcCompanion.Presentation.Hosting`.
  - `SourceStyleTests` (Roslyn `CSharpSyntaxTree.ParseText` over every `*.cs` under `src/` and `tests/`, skipping `bin/` and `obj/`), each collecting all offending files and asserting the list is empty so one run reports every file:
    - `SourceFiles_All_UseBlockScopedNamespaces`: no `FileScopedNamespaceDeclarationSyntax`.
    - `SourceFiles_All_DeclareAtMostOnePublicTopLevelTypeNamedLikeTheFile`: at most one public top-level type; if present, its identifier equals the file name with `.cs` and any `.xaml` suffix removed.
    - `SourceFiles_All_AvoidRegions`: no `RegionDirectiveTriviaSyntax`.
- [ ] **Step 2: Run** `dotnet test --project tests/EmcCompanion.Architecture.Tests`. Expected: all pass against the M0.2 code (these rules guard code that already conforms; Step 3 proves they fail when broken). If one fails, fix the production code, not the rule.
- [ ] **Step 3: Prove the rules bite.** Temporarily change `DomainAssemblyMarker.cs` to a file-scoped namespace: `dotnet build -c Release` fails with IDE0161 and `SourceFiles_All_UseBlockScopedNamespaces` fails naming the file. Revert.
- [ ] **Step 4: Commit**: `Add architecture tests for references, layers and source style`.

### Task 13: CI workflow

**Files:**
- Create: `.github/workflows/ci.yml`

**Interfaces:**
- Produces: check names `build` and `main-source` (the required checks in Task 14).

- [ ] **Step 1: Resolve action SHAs** for the latest releases of `actions/checkout` and `actions/setup-dotnet` (read-only `gh api repos/actions/<name>/releases/latest` and `.../git/ref/tags/<tag>`); pin `uses:` to the SHA with the tag in a comment.
- [ ] **Step 2: Write `ci.yml`** per spec 7.1: name `CI`; triggers; `permissions: contents: read`; concurrency `ci-${{ github.ref }}` cancel in progress. Job `build` (`windows-latest`): checkout `fetch-depth: 0`; setup-dotnet `global-json-file: global.json`; `dotnet restore --locked-mode`; `dotnet format --verify-no-changes --no-restore`; `dotnet build -c Release --no-restore`; `dotnet test -c Release --no-build`; Bridge bitness step runs `src\EmcCompanion.Bridge\bin\Release\net48\EmcCompanion.Bridge.exe` and fails unless the output contains `Process 64-bit`; gitleaks step downloads `gitleaks_8.30.1_windows_x64.zip` and `gitleaks_8.30.1_checksums.txt` from the v8.30.1 release, fails on SHA-256 mismatch, runs `gitleaks git --redact --exit-code 1 .`. Job `main-source` (`ubuntu-latest`, `if: github.event_name == 'pull_request' && github.base_ref == 'main'`): fails unless `github.head_ref` is `main-dev` or starts with `release/`.
- [ ] **Step 3: Commit**: `Add the CI workflow`.
- [ ] **Step 4: Verify** by pushing the branch and opening a draft PR (owner approves the `gh pr create --draft`): `build` is green; `main-source` does not run.

### Task 14: Repo automation and required checks

**Files:**
- Create: `.github/pull_request_template.md`, `.github/ISSUE_TEMPLATE/bug_report.yml`, `.github/ISSUE_TEMPLATE/feature_request.yml`, `.github/ISSUE_TEMPLATE/config.yml`, `.github/dependabot.yml`

- [ ] **Step 1: PR template** = charter 15.3 with a `Trello card:` line under Batch.
- [ ] **Step 2: Issue forms**: bug (steps, expected, actual, version/commit, theme and mode, confirmation checkbox "No real enterprise data, credentials or hostnames"); feature (problem, proposal, EMC parity reference); `config.yml` disables blank issues and links security reports to private vulnerability reporting.
- [ ] **Step 3: `dependabot.yml`**: ecosystems `nuget` (directory `/`) and `github-actions` (directory `/`), weekly, one group each for minor and patch updates, target branch `main-dev`.
- [ ] **Step 4: Commit**: `Add PR and issue templates and Dependabot config`.
- [ ] **Step 5: Labels** (owner approves the command list): delete GitHub's default labels; create the charter 15.7 labels with `gh label create <name> --color <hex> --description <text> --force`.
- [ ] **Step 6: Rulesets phase 2** (owner approves): add a `required_status_checks` rule with context `build` (GitHub Actions integration) to the `main-dev` ruleset, and `build` plus `main-source` to the `main` ruleset (`gh api -X PUT repos/NyxionSoftware/EMC-Companion/rulesets/<id> --input <json>`).
- [ ] **Step 7: Verify spec 7 "What to test" items 2-4** on scratch branches with draft PRs, then close them and delete the branches.
- [ ] **Step 8: Ship M0.3**: mark the PR ready, body with spec 7 "What to test", Trello card to In Review then Testing; wait for "Approved" and the squash-merge.

---

# Batch M0.4: Docs baseline

Branch `feature/m0-4-docs` from the updated `main-dev`. Trello card to In Progress.

### Task 15: Project docs

**Files:**
- Create: `docs/VERSIONING.md`, `docs/ROADMAP.md`, `docs/ARCHITECTURE.md`, `TESTING.md`, `CHANGELOG.md`

- [ ] **Step 1: `docs/VERSIONING.md`**: charter 15.4 rules; lineage table (EMC Export Utility 0.7.1; private .NET 8 EMC Companion, now the private `EMC-Companion-Legacy`; public EMC Companion from 0.1.0); pre-release format `X.Y.Z-preview.N` from `main-dev`.
- [ ] **Step 2: `docs/ROADMAP.md`**: the charter 16 milestone table plus a Status column (M0 In progress, others Planned) and a link to PROJECT.md section 16 for scope.
- [ ] **Step 3: `docs/ARCHITECTURE.md`**: one page: process model (charter 6.4), project table (6.2), the mermaid dependency graph (6.3), the six dependency rules, and which architecture test enforces each.
- [ ] **Step 4: `TESTING.md`**: how to run tests (`dotnet test`), test projects and what each covers, the hidden-UI rule (charter 14.2), the test item rule, the `LiveEmc` trait rule, the charter 14.5 template, and the "What to test" lists for M0.1-M0.4 from the spec.
- [ ] **Step 5: `CHANGELOG.md`**: Keep a Changelog header; `## [Unreleased]` with an `### Added` list written for administrators (public repo and policies, solution skeleton, CI checks, docs).
- [ ] **Step 6: Verify links**:
  Run: `Get-ChildItem . -Recurse -Filter *.md -Exclude node_modules | ForEach-Object { $f = $_; [regex]::Matches((Get-Content $f.FullName -Raw), '\]\(([^)#:]+)') | ForEach-Object { if (-not (Test-Path (Join-Path $f.DirectoryName $_.Groups[1].Value))) { "$($f.Name) -> $($_.Groups[1].Value)" } } }`
  Expected: no output.
- [ ] **Step 7: Commit**: `Add versioning, roadmap, architecture, testing docs and changelog`.

### Task 16: Clean-room EMC reference

**Files:**
- Create: `docs/emc/EMC-REFERENCE.md`

- [ ] **Step 1: Write from REA pass 01 and legacy behavior knowledge**, in our own words, with these sections: Purpose and clean-room note; Hierarchy (levels, inheritance, overrides, best practices); EMC layout (Locations tree, module tabs, table and form view, toolbar); Session model (one static session per process, so one Bridge per sign-in); Sign-in flow as user-facing states (credentials, MFA one-time password, email enrollment, password change required, server unreachable, wrong credentials) and the order EMC runs them; Self-update prohibition; Runtime configuration the Bridge reproduces (the settings and redirect list from Task 8, as behavior); Process and bitness facts (spec section 3); Menu items model (master, definition up to 64, price up to 8, availability rule); Interop names the Bridge will call, at type and member-name level only (`EMCClient.Login`, `EMCClient.ValidateOTP`, `HierarchyManager.Read`, `MenuItemData.ReadFromDb`, `LookupManager.Read`); Public sources (links to Oracle's public documentation only).
- [ ] **Step 2: Verify clean-room** (charter 2.2):
  Run: `Select-String -Path docs\emc\EMC-REFERENCE.md -Pattern '0x0[0-9a-fA-F]{7}','\b06[0-9A-F]{6}\b',$env:COMPUTERNAME,'C:\\Users\\','ldarg','callvirt','\bIL_'`
  Expected: no output. Then read it once more for real enterprise, property or person names and remove any.
- [ ] **Step 3: Commit**: `Add the clean-room EMC reference`.
- [ ] **Step 4: Ship M0.4**: self-review, push, PR (owner approves `gh pr create`), spec 8 "What to test", Trello card to In Review then Testing; wait for "Approved" and the squash-merge.

### Task 17: Release v0.1.0

Only after the owner approves the milestone.

- [ ] **Step 1:** On a `release/0.1.0` branch from `main-dev`, move CHANGELOG `[Unreleased]` to `## [0.1.0] - <date>` and add an empty `[Unreleased]`; PR into `main-dev` (squash).
- [ ] **Step 2:** PR `main-dev` into `main` titled "Release v0.1.0"; `build` and `main-source` green; the owner merges with a merge commit.
- [ ] **Step 3:** With owner approval, tag `v0.1.0` on `main` and `gh release create v0.1.0 --title "EMC Companion 0.1.0" --notes-file <CHANGELOG 0.1.0 section>`. No binary assets (the release workflow arrives in M10.4).
- [ ] **Step 4:** Trello: M0 cards and the M0 milestone card to Released.
