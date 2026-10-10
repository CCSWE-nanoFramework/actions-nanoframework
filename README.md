# actions-nanoframework

Reusable GitHub Actions workflows and composite actions for building, testing, and publishing .NET nanoFramework repos.

## Workflows

### `nanoframework-build-publish.yml`

```yaml
name: Build, test, and publish

on:
  push:
    branches: [master]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  pipeline:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/nanoframework-build-publish.yml@master
    with:
      solution: MySolution.sln
      publish-nuget: true
    secrets:
      NUGET_ORG_API_KEY: ${{ secrets.NUGET_ORG_API_KEY }}
```

| Job | Runs | Does |
|-----|------|------|
| `build` | always | Locked restore (when `packages.lock.json` files are tracked), Release build, unit tests via [vstest-nanoframework](https://github.com/CCSWE-nanoFramework/vstest-nanoframework), and with `publish-nuget` packs every tracked `*.nuspec` |
| `package-lock` | PRs | Every `.nfproj` in the solution has a `packages.lock.json` |
| `packages-updated` | PRs | `nuget update` leaves the tree unchanged |
| `publish` | push to the default branch with `publish-nuget` | Pushes the packages to nuget.org |

| Input | Default | Description |
|-------|---------|-------------|
| `solution` | | Path to the `.sln` file |
| `publish-nuget` | `false` | Pack and publish NuGet packages |

Secrets: `NUGET_ORG_API_KEY` (only for `publish`). Versions come from [Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning) when the repo has a `version.json`. Publishing requires one.

Checks render as `pipeline / build`, `pipeline / package-lock`, `pipeline / packages-updated` and `pipeline / publish`.

### `nanoframework-update-dependencies.yml`

Updates nanoFramework NuGet packages with [nanodu](https://github.com/nanoframework/nanodu) and opens a PR as the `andy-the-messenger-robot` App, so the PR triggers checks. It then closes older update PRs as superseded.

```yaml
name: Update dependencies

on:
  schedule:
    - cron: '30 20 * * *'
  repository_dispatch:
    types: update-dependencies
  workflow_dispatch:

permissions:
  contents: read

jobs:
  update-dependencies:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/nanoframework-update-dependencies.yml@master
    with:
      solution: MySolution.sln
    secrets:
      AUTOMATION_APP_ID: ${{ secrets.AUTOMATION_APP_ID }}
      AUTOMATION_APP_KEY: ${{ secrets.AUTOMATION_APP_KEY }}
```

Secrets: `AUTOMATION_APP_ID`, `AUTOMATION_APP_KEY`. Pass them explicitly: `secrets: inherit` doesn't cross organizations. These are org secrets in CCSWE-nanoFramework. Repos outside the org need them as repo secrets, stored in `midworld-internal/secrets` at `github/andy-the-messenger-robot.yaml`.

## Composite actions

Used by the workflows and by `vstest-nanoframework`'s CI. Reference them at `@master`.

### `setup-nanoframework`

Installs the nanoFramework build components, MSBuild (x64) and NuGet CLI 7.x.

```yaml
- uses: CCSWE-nanoFramework/actions-nanoframework/.github/actions/setup-nanoframework@master
```

### `build-nanoframework`

Restores and builds a solution in Release. The restore is locked when `packages.lock.json` files are tracked. When a `version.json` exists, it runs Nerdbank.GitVersioning and stamps the versions; the `NBGV_*` variables stay available to later steps.

```yaml
- uses: CCSWE-nanoFramework/actions-nanoframework/.github/actions/build-nanoframework@master
  with:
    solution: MySolution.sln
```
