# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Guidelines

- Do not include `Co-Authored-By` or any AI attribution in commit messages.
- When staging for a commit, use `git add -A` but flag any changes that appear
  unrelated to the current task and ask whether to include them.
  
## What This Repository Is

This is a **GitHub Actions reusable workflow library** for the [CCSWE-nanoFramework](https://github.com/CCSWE-nanoFramework) organization. It contains no application source code — only three reusable workflow definitions that other nanoFramework repositories call via GitHub's `workflow_call` trigger.

## Repository Structure

```
.github/workflows/
├── build-solution.yml       # Core reusable workflow: build, test, and publish NuGet
├── pull-request-checks.yml  # PR validation: delegates to nanoframework/nf-tools
└── update-dependencies.yml  # Scheduled dependency updates: delegates to nanoframework/nf-tools
```

## Workflow Architecture

All three workflows use `on: workflow_call` — they are not triggered directly, but called by consuming repositories like:

```yaml
jobs:
  build:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/build-solution.yml@master
    with:
      solution: MySolution.sln
    secrets: inherit
```

### build-solution.yml

The main workflow. Inputs:
- `solution` (required): path to `.sln` file
- `publishNuGet` (boolean, default `true`): build and publish NuGet packages
- `runUnitTests` (boolean, default `true`): run unit tests via `vstest-nanoframework`

Pipeline steps:
1. Checkout with `fetch-depth: 0` (required for Nerdbank.GitVersioning)
2. `nanoframework/nanobuild@v1` — sets up nanoFramework build environment
3. `dotnet tool update -g nanoclr` — updates the nanoclr tool
4. `microsoft/setup-msbuild@v3` (x64)
5. `nuget/setup-nuget@v3` + restore
6. `dotnet/nbgv@v0.5.1` — semantic versioning via Nerdbank.GitVersioning
7. `msbuild` with version parameters injected from NBGV
8. `CCSWE-nanoFramework/vstest-nanoframework@v1` (if `runUnitTests`)
9. Build NuGet packages from `.nuspec` files (if `publishNuGet`)
10. Publish to nuget.org (only on `master` branch, requires `NUGET_ORG_API_KEY` secret)

Key environment: `BUILD_CONFIGURATION=Release`, runs on `windows-latest` with PowerShell (`pwsh`).

### pull-request-checks.yml

Delegates entirely to external workflows in `nanoframework/nf-tools`:
- `check-package-lock.yml` — validates package lock files
- `check-packages-updated.yml` — ensures packages are up to date

### update-dependencies.yml

Inputs: `solution` (required), `branchToPr` (default: `master`). Delegates to `nanoframework/nf-tools/.github/workflows/update-dependencies.yml@main`.

## Key Secrets

- `GITHUB_TOKEN` — standard GitHub token passed via `secrets: inherit`
- `NUGET_ORG_API_KEY` — required for publishing packages to nuget.org (only used on `master` branch builds)

## Making Changes

Since there is no build/test system in this repo itself, changes are validated by observing consuming repositories' workflow runs. When modifying workflows:

- All three workflows are called with `@master`, so changes take effect immediately for all consumers
- Test changes by checking a downstream repository's Actions tab after pushing
- The `build-solution.yml` step order matters — MSBuild setup must precede NuGet restore, and NBGV must run before the `msbuild` call so version variables are available
