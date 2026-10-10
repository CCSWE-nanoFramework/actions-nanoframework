# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Guidelines

- Do not include `Co-Authored-By` or any AI attribution in commit messages.
- When staging for a commit, use `git add -A` but flag any changes that appear
  unrelated to the current task and ask whether to include them.

## What This Repository Is

Reusable GitHub Actions workflows (`workflow_call`) and composite actions for CCSWE-nanoFramework repos. No application code. See README for the caller contract.

```
.github/workflows/nanoframework-build-publish.yml        build, PR checks (package-lock, packages-updated), publish
.github/workflows/nanoframework-update-dependencies.yml  nanodu updates as the andy-the-messenger-robot App, closes superseded PRs
.github/actions/setup-nanoframework/action.yml           nanobuild, MSBuild (x64), NuGet CLI 7.x
.github/actions/build-nanoframework/action.yml           NBGV (when version.json exists), restore, MSBuild Release
```

## Conventions

- Jobs use bash; Windows only where a tool needs it (msbuild, `nuget update`, nanodu).
- Inputs reach `run:` only through `env:`.
- Every job declares least-privilege `permissions:`; reusable workflows declare their secrets and never forward `secrets: inherit` to third parties.
- Pin actions to their latest major tag.
- Callers' required checks are `pipeline / build`, `pipeline / package-lock` and `pipeline / packages-updated`. Renaming jobs breaks branch protection and rulesets in caller repos.

## Making Changes

- Validate with `actionlint .github/workflows/*.yml`.
- Callers use `@master`, and the workflows reference the composite actions at `@master`, so merged changes take effect immediately. Verify on a caller's next run (CCSWE.nanoFramework, Emily.Clock) or `vstest-nanoframework`'s CI.
- Step order matters: `setup-nanoframework` before `build-nanoframework`, and NBGV before msbuild.
