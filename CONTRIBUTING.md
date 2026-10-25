# Contributing to EMC Companion

The project charter, [docs/PROJECT.md](docs/PROJECT.md), is the single source of truth for scope, rules, and architecture. This guide pulls out what you need before your first pull request. Read the clean-room rule first. It binds every contributor and every automated agent. When in doubt, do not commit.

## Workflow

Work proceeds in small batches (charter [15.1](docs/PROJECT.md#151-small-batch-process)).

1. The coordinator or agent picks the next batch from the Up Next list on the Trello board (charter [section 16](docs/PROJECT.md#16-roadmap)). Each card is one batch of 5-10 tasks with one theme.
2. Work on a short-lived feature branch such as `feature/m4-2-mim-inline-edit`, created from `main-dev`.
3. Open one PR per batch into `main-dev`, with the PR template filled in (summary, tasks, screenshots from the harness, What to test, checklist).
4. CI must be green.
5. The owner tests on the owner's workstation and replies **"Approved"** or sends a bug list.
6. Fix bugs on the same branch (new commits) until the batch is Approved.
7. Squash-merge into `main-dev`. Retarget any dependent PRs to `main-dev` immediately.
8. Update the CHANGELOG `[Unreleased]` section in the same PR.
9. When a milestone is complete, cut a preview (optional) or a release (charter [15.5](docs/PROJECT.md#155-release-steps)).

## Branches

| Branch | Purpose | Protection |
|---|---|---|
| `main` | Stable, released code; every commit tagged `vX.Y.Z` | Protected; PR only from `main-dev`; required checks; no force push |
| `main-dev` | Integration and owner testing | Protected; PR required; required checks |
| `feature/*`, `fix/*`, `docs/*` | Short-lived batch work | None; deleted after merge |
| `release/X.Y.Z` | Optional, for release stabilization | Temporary |

## Code style

The coding standards are in charter [section 7](docs/PROJECT.md#7-coding-standards). The `.editorconfig` at the repository root encodes the rules that the compiler and `dotnet format` can check. Write code as if the owner wrote it: consistency beats personal preference.

## Clean-room rule

This is charter [section 2.2](docs/PROJECT.md#22-clean-room-rule-for-rea-reverse-engineer-anything). The owner uses REA (the owner's local REA tooling) to decompile and deeply understand EMC's behavior and UX. The output of that work is **knowledge**, never code.

Rules:

1. REA runs **only on the owner's workstation** against the owner's licensed dev copy at `%USERPROFILE%\source\repos\Oracle Hospitality\EMCClient\`.
2. Decompiled output stays **outside the repository** (for example `%USERPROFILE%\REA-Notes\`), never in a cloned working tree, never in a scratch folder inside the repo.
3. Notes that enter the repo (`docs/emc/`) **describe behavior**: what a screen shows, which fields exist, what validation happens, what order operations run in, what EMC refuses and why, which API surface (type and member names) the Bridge must call. They never paste decompiled method bodies, IL, or resource contents.
4. Type and member names that the Bridge must call are interoperability facts and may be recorded (for example "the menu item master collection is loaded through the configuration service by object number range"). Implementation details are rewritten in our own words.
5. Implementation in EMC Companion is written fresh, in the owner's style, from the behavior notes. Whoever writes Bridge code should work from `docs/emc/` notes, not from an open decompiler window.
6. Every PR that touches the Bridge or adds EMC behavior includes the checkbox: "No decompiled code, Oracle assets, or customer data are included."
7. If a contributor is unsure whether something is behavior or code, it is treated as code and stays out.

## Sample data only

Screenshots, tests, docs, and the screenshot harness use a fictional dataset (`samples/Contoso Grill` or similar, see [14.4](docs/PROJECT.md#144-sample-dataset)). Names, object numbers, and prices are invented. Real enterprise names, property names, employees, or server names are never used, including in commit messages, issue text, or PR descriptions. This is charter [section 2.5](docs/PROJECT.md#25-sample-data-only).

## Commit messages

From charter [section 7.5](docs/PROJECT.md#75-commit-messages):

- Imperative, short subject (max about 72 chars), optional scope: `MIM: keep object number when switching to Definitions`.
- Body explains why when non-obvious; references issues (`Refs #12`, `Fixes #14`).
- One logical change per commit within a batch PR. PRs are squash-merged into `main-dev` with the batch title.
- Attribution trailer lines for AI-assisted commits as configured by the owner's tooling.

## Secret scanning

This repository is public, so no secret may ever land in it (charter [section 2.4](docs/PROJECT.md#24-public-repo-secret-hygiene)). Install the `pre-commit` hook so that gitleaks checks every commit on your machine. It needs Python:

```powershell
pip install pre-commit
pre-commit install
```

The hook is configured in `.pre-commit-config.yaml`. CI runs gitleaks again as a backstop. Test placeholders must not look like real secrets: use `example.invalid` hosts and obviously fake values like `not-a-password`.

To check that the hook works, generate a throwaway fake token, try to commit it,
and confirm the commit is blocked. Never push it.

```powershell
$fake = 'ghp_' + -join ((48..57) + (65..90) + (97..122) | Get-Random -Count 36 | ForEach-Object { [char]$_ })
Set-Content -Path hook-test.txt -Value "token = $fake"
git add hook-test.txt
git commit -m "hook test"   # expected: blocked by gitleaks
git reset hook-test.txt; Remove-Item hook-test.txt
```

Run this on a scratch branch: `git switch -c hook-test` first. If the commit is blocked, the hook works; switch back with `git switch -` and delete the branch with `git branch -D hook-test`. If the commit succeeds, the hook is not working: run `git switch -` and `git branch -D hook-test`, which discards the fake commit, then fix your hook setup. Never push that branch.
