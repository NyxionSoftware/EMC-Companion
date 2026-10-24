# M0 Bootstrap: design spec

| Item | Value |
|---|---|
| Milestone | M0, repo bootstrap, architecture skeleton, CI (target version 0.1.0) |
| Status | Draft for owner review |
| Date | 2026-10-24 |
| Source of truth | `docs/PROJECT.md` (the project charter, added in M0.1) |

This spec turns charter sections 2, 6, 7, 15 and 16 (M0) into concrete, testable
work. Where the charter already says how something works, this spec points to it
instead of repeating it. Where this spec and the charter disagree, the spec records
the newer decision and M0.1 updates the charter to match.

## 1. Goal and success criteria

M0 delivers a public, clean, source-only repository with the full layered solution
skeleton, CI that enforces the coding and clean-room rules, and the baseline docs.
Nothing in M0 talks to EMC.

M0 is done when:

1. `NyxionSoftware/EMC-Companion` is public, its history contains no legacy commits,
   and `main` and `main-dev` are protected.
2. `dotnet build` and `dotnet test` pass on a clean Windows machine with only the
   .NET 10 SDK installed (the .NET Framework 4.8 reference assemblies come from NuGet).
3. The app opens an empty themed-neutral shell window and closes cleanly; the Bridge
   runs as a 64-bit .NET Framework process and prints its version and bitness.
4. CI fails on a file-scoped namespace, a forbidden project reference, a tracked
   binary or build folder, a formatting violation, or a secret pattern.
5. The owner approves each of the four batch PRs (M0.1 to M0.4).

## 2. Decisions recorded

Charter section 17 recommendations are adopted unless listed otherwise. M0.1 writes
these outcomes into the charter's "Owner decision" column.

| # | Decision | Outcome |
|---|---|---|
| D1 | License | Apache-2.0 for the whole repo, including Plugins.Abstractions and Bridge.Contracts |
| D2 | .NET deployment | Self-contained (applies from M11) |
| D3 | Code signing | SignPath Foundation if eligible, else Azure Trusted Signing (applies from M11) |
| D4 | IPC serializer | System.Text.Json first, measured; see risk R3 |
| D5 | Bridge EMC binding | Runtime reflection behind small typed wrappers |
| D6 | Bridge target | net48 |
| D7 | Glass rendering | DWM Mica/Acrylic on Windows 11, static blurred backdrop on Windows 10 |
| D8 | Plugin trust | Unsigned allowed with a warning; a future gallery requires hashes |
| D9 | Fonts | Source Serif 4, Inter, JetBrains Mono (all SIL OFL) |
| D10 | Old history | Fresh start; legacy behavior is ported by batch after clean-room review |
| D11 | Starting version | 0.1.0, lineage documented in `docs/VERSIONING.md` |
| D12 | Repository name | Old repo renamed `EMC-Companion-Legacy` (private); new public `EMC-Companion` |
| D13 | Install path | Keep `C:\Program Files\Oracle Hospitality\EMC Companion\` with clear branding |
| D14 | Grid | Styled WPF DataGrid |
| D15 | Assertions | Shouldly |
| D16 | Preview channel | Ships at v1, off by default |
| D17 | Machine plugins | Allowed in `{app}\plugins`, read-only for users |
| D18 | Trello visibility | Private board |
| D19 | Agents and Trello | Agents move cards to In Progress and In Review and comment; only the owner moves cards out of Testing |
| D20 | Trello Power-Ups | GitHub Power-Up plus Butler; Custom Fields if the plan allows |
| D21 | Bridge bitness (new) | x64 by default, matching how EMC.exe runs; `BridgePlatform=x86` build switch kept until M1.3 verifies x64 with a live sign-in |
| D22 | Repo visibility (new) | New repo is public from creation |
| D23 | Branch content (new) | Source only; no build output or binaries in any branch. Release binaries are built by CI and attached to GitHub Releases |
| D24 | Charter location (new) | The charter lives in the repo as `docs/PROJECT.md`, scrubbed of environment details, and is updated by PR like any doc |
| D25 | Solution format (new) | `EmcCompanion.slnx` (.NET 10 default; the owner's Visual Studio 2022 17.14 and 2026 18.10 both open it) |

## 3. Corrections from REA pass 01

REA pass 01 (owner-local notes, never committed) inspected the owner's licensed EMC
19.10.2.0.27 dev copy with execution-free metadata inspection. M0.1 corrects the
charter with these behavior-level facts:

| Charter text | Correction |
|---|---|
| 1.1, 3.4: EMC client is 32-bit, DevExpress 17.1 | EMC.exe is an AnyCPU .NET Framework 4.6.2 executable and runs 64-bit on x64 Windows. The 32-bit, DevExpress 17.1 tool in the folder is LDSConfigurator, not EMC. EMC's UI is WinForms |
| 3.4: "This is why the Bridge exists: a 64-bit process cannot load 32-bit assemblies" | The Bridge exists because EMC's libraries are .NET Framework assemblies with WinForms and web-service dependencies that .NET 10 cannot host. Bitness is not the reason |
| 8.4: EMC is a DevExpress WinForms-era tool | EMC is a WinForms tool; keep the described look (dense, grey, toolbar, blue selection) |
| 6.4: Bridge config with EMC's binding redirects | Confirmed. EMC expects redirects for Newtonsoft.Json, System.Memory, System.Buffers, System.Runtime.CompilerServices.Unsafe, System.Text.Json (8.0.0.5), System.Formats.Asn1 and System.Numerics.Vectors, plus legacy path handling and corrupted-state exception policy. The exact list goes in `docs/emc/EMC-REFERENCE.md` |
| New rule | EMC's sign-in flow includes a self-update step that rewrites the EMC install folder. The Bridge never runs it and keeps `PerformEMCVersionCheck` off |
| New fact | EMC's session client is static, so one Bridge process holds exactly one EMC session (matches 6.4) |

## 4. Repository and branches

### 4.1 Restructure steps (done once, before M0.1's PR)

1. Backup bundle of the old repo (all heads and the 56 pull refs) kept at
   `%USERPROFILE%\source\repos\EMC-Companion-legacy-backup-2026-10-24.bundle`. Done.
2. Rename `NyxionSoftware/EMC-Companion` to `NyxionSoftware/EMC-Companion-Legacy`.
   It stays private and keeps its pull requests and history.
3. In the legacy repo: create `legacy` at the old `main-dev` tip (`b166aeb`), make it
   the default branch, delete every other branch.
4. Create a new public `NyxionSoftware/EMC-Companion` with no auto-generated files.
5. Push the local root commit (`Initial commit`, a three-line README) as both `main`
   and `main-dev`. Set `main-dev` as the default branch so PRs target it by default.
6. Enable secret scanning, push protection, Dependabot alerts and security updates,
   and private vulnerability reporting.
7. Apply the rulesets in 4.2.

Commits use the owner's GitHub no-reply identity. Hostnames never appear in author
fields (the legacy history has 67 commits with a hostname in the author email, one
more reason it stays private).

### 4.2 Branch rules

| Branch | Rules |
|---|---|
| `main` | Ruleset: PR required (0 approvals, the owner is the only maintainer), required checks `build` and `main-source`, no force push, no deletion. `main-source` fails any PR into `main` whose head is not `main-dev` or `release/*` |
| `main-dev` | Ruleset: PR required (0 approvals), required check `build`, no force push, no deletion |
| `feature/*`, `fix/*`, `docs/*` | No rules; deleted after merge (repo setting "Automatically delete head branches") |

PRs into `main-dev` are squash-merged; release PRs into `main` use a merge commit
(charter 15.5).

### 4.3 Source-only policy

- `.gitignore` covers charter 2.4 and 19.5 plus `bin/`, `obj/`, `.vs/`, `TestResults/`.
- `.gitattributes`: `* text=auto eol=crlf` for source, `binary` for `*.png`, `*.ico`,
  `*.ttf`, `*.otf`.
- CI job step `source-only` fails if any tracked file matches `*.dll`, `*.exe`,
  `*.pdb`, `*.nupkg`, `*.snupkg`, `*.zip`, `*.msi`, `*.7z`, or lives under a `bin/`,
  `obj/`, `artifacts/` or `publish/` folder. Allowed binary sources are images and
  fonts under `assets/` and images under `docs/`.
- Users build from source. CI builds release binaries only on `v*` tags (M10.4).

## 5. M0.1 Repository and legal

Branch `feature/m0-1-repo-legal`. This spec and the M0 plan are its first commits.

Tasks:

1. `docs/PROJECT.md`: the charter, scrubbed and updated (rules below).
2. `LICENSE` (Apache-2.0 text), `NOTICE` (charter 2.6), `THIRD-PARTY-NOTICES.md`
   (table headings only; each batch that adds a shipped package adds its row, starting
   with M0.2's Microsoft.Extensions.Hosting and CommunityToolkit.Mvvm).
3. `README.md`: name, one-paragraph description, status badges (version, .NET 10,
   platform, license), "not affiliated with Oracle" notice, build-from-source steps,
   links to PROJECT.md and CONTRIBUTING.
4. `CONTRIBUTING.md`: workflow (charter 15.1), branch rules, style (charter 7), the
   clean-room rule (charter 2.2) in full, sample-data-only rule, commit format.
5. `SECURITY.md`: private vulnerability reporting through GitHub, scope, what not to
   attach (logs with real data).
6. `.gitignore`, `.gitattributes`, `.editorconfig` (section 6.4),
   `.pre-commit-config.yaml` (gitleaks pinned; ggshield optional, documented).
7. `CODEOWNERS` (`* @NyxionSoftware`).

Charter scrub and update rules for `docs/PROJECT.md`:

- Replace the workstation hostname with "the owner's workstation".
- Replace the owner's user-profile paths (`C:\Users\<name>\`) with `%USERPROFILE%\`.
- Keep the REA rules but describe REA tooling generically (no local install details).
- Fill section 17's "Owner decision" column from section 2 of this spec; add D21-D25.
- Apply the section 3 corrections in place (1.1, 3.4, 6.4, 8.4).
- Update 1.1 "Repository" and 3 "Accomplishments" to say the legacy repo is
  `EMC-Companion-Legacy` (private).
- Update 6.1 to `EmcCompanion.slnx`.
- No other content changes; later edits go through normal PRs.

What to test (owner):

1. Open the repo on GitHub: README shows the Oracle notice and build steps.
2. `docs/PROJECT.md` has no workstation hostname and no `C:\Users\<name>` path
   (search the file).
3. Install the pre-commit hook (`pre-commit install`), then try to commit a file
   containing a fake token produced by the generator command in TESTING.md: the
   gitleaks hook blocks the commit. Nothing secret-shaped is ever pushed (charter 3.3
   lesson).
4. Repo Settings > Code security: secret scanning and push protection show Enabled.
5. LICENSE is Apache-2.0; NOTICE matches charter 2.6.

## 6. M0.2 Solution skeleton

Branch `feature/m0-2-solution-skeleton`.

### 6.1 Projects

| Project | Target | Output | Contents in M0 |
|---|---|---|---|
| `src/EmcCompanion.Domain` | net10.0 | library | Empty except an assembly marker type for architecture tests |
| `src/EmcCompanion.Application` | net10.0 | library | `AddApplication()` extension (registers nothing yet) |
| `src/EmcCompanion.Infrastructure` | net10.0-windows | library | `AddInfrastructure()` extension |
| `src/EmcCompanion.Presentation` | net10.0-windows, WPF | `EmcCompanion.exe` | `App`, Generic Host composition root in `Hosting/`, `AddPresentation()`, empty `ShellWindow` bound to `ShellViewModel` (CommunityToolkit.Mvvm) whose `Title` is "EMC Companion" |
| `src/EmcCompanion.Bridge` | net48, x64 | `EmcCompanion.Bridge.exe` | `Program` that prints product, version, contract version and process bitness, exits 0; `App.config` with EMC's runtime settings (section 3) |
| `src/EmcCompanion.Bridge.Contracts` | netstandard2.0 | library | `ContractInfo.Version = 1` |
| `src/EmcCompanion.Plugins.Abstractions` | net10.0 | library | `PluginApi.Version = "1.0"` |
| `src/EmcCompanion.Updater` | net10.0-windows | library | `AddUpdater()` extension |
| `tests/EmcCompanion.Architecture.Tests` | net10.0-windows | xUnit v3 | Rules in 6.3 |
| `tests/EmcCompanion.Bridge.Contracts.Tests` | net10.0 | xUnit v3 | Contract version is 1 |
| `tests/EmcCompanion.Presentation.Tests` | net10.0-windows | xUnit v3 | Host builds with `ValidateOnBuild` and `ValidateScopes`; `ShellViewModel` resolves with its title (no window is created) |

Deviation from charter 6.1: test projects are added with their first real test, so
there are no empty test projects (xUnit v3 on Microsoft Testing Platform fails a run
with zero tests). Domain, Application and Infrastructure tests and `EmcCompanion.Fakes`
arrive in M1. `tools/` arrives with the screenshot harness (M2.5).

### 6.2 Shared build files

- `global.json`: SDK `10.0.401`, `rollForward: latestFeature`, and
  `test.runner: Microsoft.Testing.Platform` so `dotnet test` runs xUnit v3 natively.
- `nuget.config`: clears sources, adds nuget.org only.
- `Directory.Build.props`: charter 19.4 plus `RestorePackagesWithLockFile`,
  `Deterministic`, `EnforceCodeStyleInBuild`, `AnalysisLevel latest-recommended`,
  `GenerateDocumentationFile` for Plugins.Abstractions and Bridge.Contracts, and
  `Authors` "Jamie Head" / `Company` "NyxionSoftware" as in the legacy props.
- `Directory.Packages.props`: central versions for every package.
- Bridge: `Microsoft.NETFramework.ReferenceAssemblies` so it builds without Visual
  Studio's targeting pack; `PlatformTarget` from `$(BridgePlatform)`, default `x64`.
- `EmcCompanion.slnx` with `src` and `tests` solution folders.

### 6.3 Composition root

`App.OnStartup` builds a Generic Host (`Host.CreateApplicationBuilder`), calls
`AddApplication()`, `AddInfrastructure()`, `AddUpdater()`, `AddPresentation()`,
starts it, resolves `ShellWindow` and shows it. `OnExit` stops and disposes the host.
Logging uses `Microsoft.Extensions.Logging` with the debug provider only; Serilog
arrives in M3.5. No EMC code paths exist in M0.

### 6.4 `.editorconfig`

Starts from the legacy `.editorconfig` (block namespaces, Allman braces, sorted
System usings) and raises these to `error`: `IDE0161` (block-scoped namespaces),
`IDE0011` (braces), `IDE0005` (unused usings), `IDE0040` (explicit accessibility),
`IDE0044` (readonly fields), `IDE0065` with
`csharp_using_directive_placement = outside_namespace` (Visual Studio default usings).
Sets `dotnet_separate_import_directive_groups = true` so System, project and
third-party usings form separate groups as in charter 7.3. Adds naming rules for
`_camelCase` private fields, `I`-prefixed interfaces, `Async` suffix, and PascalCase
constants (charter 7.2).

### 6.5 Local deploy script

`build/Deploy-Local.ps1` (requires an elevated PowerShell):

1. Refuses to run unless elevated.
2. Publishes the app (`win-x64`, self-contained) and the Bridge (x64) to
   `publish/local/` (ignored).
3. If `EmcCompanion.exe` is running from the target folder, asks before closing it.
4. Copies into `C:\Program Files\Oracle Hospitality\EMC Companion\` (or `-Target`),
   keeping an existing `appsettings.json`.
5. Prints the deployed version.

What to test (owner):

1. Fresh clone, `dotnet build` then `dotnet test`: both pass.
2. Run `src\EmcCompanion.Presentation\bin\Debug\net10.0-windows\EmcCompanion.exe`:
   an empty window titled "EMC Companion" opens and closes cleanly.
3. Run `EmcCompanion.Bridge.exe`: it prints version 0.1.0, contract 1, and `64-bit`.
4. Build with `-p:BridgePlatform=x86`: the Bridge prints `32-bit`.
5. In an elevated PowerShell, run `build\Deploy-Local.ps1`: the app runs from the
   Program Files folder.

## 7. M0.3 CI and architecture tests

Branch `feature/m0-3-ci`.

### 7.1 `ci.yml`

Triggers: pull requests into `main-dev` and `main`, pushes to `main-dev` and `main`.
Runner `windows-latest`. Concurrency group per ref, cancel in progress.

Job `build` steps: checkout (full history for gitleaks), setup-dotnet from
`global.json`, `source-only` guard (4.3), `dotnet restore --locked-mode`,
`dotnet format --verify-no-changes`, `dotnet build -c Release`,
`dotnet test -c Release --no-build`, run the Bridge exe and assert it reports
`64-bit`, gitleaks CLI (pinned release, checksum-verified, `detect` over the
history; run directly so no action license is needed).

Job `main-source` (only on PRs into `main`): fails unless the head branch is
`main-dev` or `release/*`.

Workflow permissions default to `contents: read`. Actions are pinned to full
commit SHAs; Dependabot keeps them current.

### 7.2 Architecture tests

1. Project references, read from each `.csproj`: the dependency graph in charter 6.3
   exactly. Bridge references only Bridge.Contracts; Plugins.Abstractions,
   Bridge.Contracts and Domain reference nothing.
2. Type dependencies (NetArchTest.eNhancedEdition): Application has no dependency on
   Infrastructure, Presentation, WPF (`System.Windows`) or Bridge.Contracts; only
   types in `EmcCompanion.Presentation.Hosting` and `App` depend on Infrastructure.
3. Source rules over `src/` and `tests/`: no file-scoped namespace, one public type
   per file with matching file name (for `Name.xaml.cs` the type is `Name`), no
   `#region`.
4. No project references a file under `lib/emc/` or any path containing `EMCClient`.

### 7.3 Repo automation

- `.github/pull_request_template.md` (charter 15.3, with the Trello card line).
- Issue forms `bug_report.yml`, `feature_request.yml`, and `config.yml` pointing
  security reports to private reporting.
- Labels from charter 15.7, created with `gh label create`.
- `.github/dependabot.yml`: NuGet and GitHub Actions, weekly, grouped minor/patch.

What to test (owner):

1. The M0.3 PR shows a green `build` check.
2. On a scratch branch, change one namespace to file-scoped and open a draft PR:
   `build` fails on the architecture test and the analyzer.
3. On a scratch branch, add an empty `test.dll`: `build` fails at `source-only`.
4. Open a PR from a feature branch into `main`: `main-source` fails.

## 8. M0.4 Docs baseline

Branch `feature/m0-4-docs`.

- `docs/VERSIONING.md`: SemVer rules (charter 15.4) and the lineage EMC Export Utility
  0.7.1, private .NET 8 EMC Companion (now EMC-Companion-Legacy), public EMC Companion
  0.1.0.
- `docs/ROADMAP.md`: milestone table with status, pointing to PROJECT.md section 16
  for scope.
- `docs/ARCHITECTURE.md`: one-page summary of charter section 6 with the dependency
  diagram; PROJECT.md stays authoritative.
- `docs/emc/EMC-REFERENCE.md`: written fresh from REA pass 01 and the legacy behavior
  knowledge. Behavior and interop names only: hierarchy model, module layout, the
  sign-in sequence and its user-facing states, session model, runtime configuration
  the Bridge must reproduce, the self-update prohibition, menu item master/definition/
  price model and limits. No metadata tokens, no method-body descriptions, no IL, no
  screenshots, no real enterprise, property or person names, no hostnames.
- `TESTING.md`: how to run tests, the hidden-UI rule, the test item rule, and the
  per-batch "What to test" template (charter 14.5) with M0's lists.
- `CHANGELOG.md`: Keep a Changelog; `[Unreleased]` lists M0 additions for administrators.

What to test (owner):

1. Docs render on GitHub with working relative links.
2. Read `docs/emc/EMC-REFERENCE.md`: it matches your understanding of EMC, contains no
   code, tokens or real names.

## 9. Trello board

After this spec is approved, create the private board "EMC Companion Roadmap" from
charter 16 "Roadmap tracking in Trello": 9 lists, milestone and type/area/priority/
warning labels, 65 batch cards with Tasks checklists, and 14 pinned milestone cards.
M0 cards go to Up Next; the rest to Backlog / Ideas. Card descriptions use the charter
template and link to `docs/PROJECT.md`. The GitHub Power-Up and Butler rules are
enabled by the owner in Trello's UI (Power-Ups need the owner's account consent).

## 10. Out of scope for M0

Bridge IPC, sign-in, EMC detection, sample data, theming, any EMC call, Serilog, the
screenshot harness, installer, updater, plugins, release workflow.

## 11. Risks and open items

| # | Risk | Mitigation |
|---|---|---|
| R1 | x64 Bridge fails with EMC libraries at runtime | `BridgePlatform=x86` switch; M1.3 live sign-in test decides; D21 revisited then |
| R2 | xUnit v3 / Microsoft Testing Platform friction with `dotnet test` on .NET 10 | Use the `Microsoft.Testing.Platform` runner mode configured in `global.json`; fall back to xUnit v2 + VSTest if CI is unstable |
| R3 | Bridge's System.Text.Json conflicts with EMC's 8.0.0.5 | Bridge pins System.Text.Json 8.0.x in M1.1 and redirects to it; if that fails, the Bridge uses the redirect target EMC ships |
| R4 | GitHub push protection does not catch the test pattern | The pre-commit gitleaks hook and the CI gitleaks step are independent backstops |
| R5 | Legacy PR content becomes public by accident | Only the renamed private repo holds it; the new repo starts from a new root commit |
