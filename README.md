# actions-nanoframework

Reusable GitHub Actions workflows for building, testing, and publishing .NET nanoFramework libraries.

## Workflows

### `build-solution.yml`

Builds a .NET nanoFramework solution, optionally runs unit tests, and publishes NuGet packages to nuget.org on `master` branch pushes.

**Usage:**

```yaml
jobs:
  build:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/build-solution.yml@master
    with:
      solution: MySolution.sln
    secrets: inherit
```

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `solution` | yes | | Path to the `.sln` file |
| `publishNuGet` | no | `true` | Build and publish NuGet packages |
| `runUnitTests` | no | `true` | Run unit tests |

**Secrets required:**

- `GITHUB_TOKEN` — provided automatically
- `NUGET_ORG_API_KEY` — required when `publishNuGet` is `true` and running on `master`

**Notes:**

- Packages are published to nuget.org only when the build runs on the `master` branch.
- Version numbers are derived automatically via [Nerdbank.GitVersioning](https://github.com/dotnet/Nerdbank.GitVersioning). A `version.json` file is required in consuming repositories.
- `.nuspec` files in the `packages` folder are excluded from NuGet packaging.

---

### `pull-request-checks.yml`

Validates pull requests by checking that package lock files are consistent and all NuGet packages are up to date.

**Usage:**

```yaml
jobs:
  pr-checks:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/pull-request-checks.yml@master
    with:
      solution: MySolution.sln
    secrets: inherit
```

**Inputs:**

| Input | Required | Description |
|-------|----------|-------------|
| `solution` | yes | Path to the `.sln` file |

---

### `update-dependencies.yml`

Checks for updated .NET nanoFramework NuGet dependencies and opens a pull request with any updates found.

**Usage:**

```yaml
on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday

jobs:
  update-dependencies:
    uses: CCSWE-nanoFramework/actions-nanoframework/.github/workflows/update-dependencies.yml@master
    with:
      solution: MySolution.sln
    secrets: inherit
```

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `solution` | yes | | Path to the `.sln` file |
| `branchToPr` | no | `master` | Target branch for the dependency update PR |
