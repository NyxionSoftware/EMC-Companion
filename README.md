# EMC Companion

![Version](https://img.shields.io/badge/version-0.1.0--dev-orange)
![.NET](https://img.shields.io/badge/.NET-10-512BD4)
![Platform](https://img.shields.io/badge/platform-Windows%2010%2B%20x64-0078D6)
![License](https://img.shields.io/badge/license-Apache--2.0-blue)
![Secret scan](https://img.shields.io/badge/secret%20scan-gitleaks-success)

EMC Companion is a passion project by NyxionSoftware: a modern, themeable, plugin-capable desktop companion for the Oracle Simphony Enterprise Management Console (EMC). A small .NET Framework **Bridge** process loads the EMC client libraries that are already installed on the user's machine, talks to Simphony on the app's behalf, and translates the results into a versioned contract that the 64-bit **.NET 10 WPF** application can read, understand, and edit.

## Status

Bootstrapping (M0). The repository holds the project charter, the legal files, and the contributor guide so far. The code follows in later M0 batches. The roadmap is in the [charter](docs/PROJECT.md#16-roadmap).

## Requirements

EMC Companion requires a licensed Oracle Simphony EMC client to do anything useful. It runs on Windows 10 or later, 64-bit.

## Build from source

Install the .NET SDK 10.0.401 or later, then run:

```powershell
git clone https://github.com/NyxionSoftware/EMC-Companion.git
cd EMC-Companion
dotnet build
dotnet test
```

No Oracle files are needed to build. The solution lands in milestone M0.2, so until then there is nothing to build yet.

Branches contain source only. Binaries come only from GitHub Releases, built by CI.

## Branches

- `main` is stable, released code.
- `main-dev` is where development happens. Open pull requests against it.

See [CONTRIBUTING.md](CONTRIBUTING.md#branches) for the full branch table.

## More

- [docs/PROJECT.md](docs/PROJECT.md): the project charter (scope, rules, architecture, roadmap).
- [CONTRIBUTING.md](CONTRIBUTING.md): how to contribute, including the clean-room rule.
- [SECURITY.md](SECURITY.md): how to report a vulnerability.
- [LICENSE](LICENSE) and [NOTICE](NOTICE): the Apache License 2.0 and the Oracle disclaimer.

## Not affiliated with Oracle

EMC Companion is an independent project. It is not affiliated with, sponsored by, or endorsed by Oracle Corporation or its affiliates.

Oracle, Simphony, and MICROS are trademarks or registered trademarks of Oracle and/or its affiliates. Other names may be trademarks of their respective owners. EMC Companion does not include, bundle, or redistribute any Oracle software.
