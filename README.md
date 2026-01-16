# pixi-lock

> [!NOTE]
> This repo is likely to be moved to https://github.com/xarray-contrib

This repo provides two GitHub Actions for managing `pixi.lock` files with caching:

- **`create-and-cache`**: Generates a `pixi.lock` file and caches it
- **`restore`**: Restores the cached `pixi.lock` file in downstream jobs

This two-action pattern is designed for CI workflows where you want to generate the lock file once and reuse it across multiple matrix jobs.


## Usage

```yaml
jobs:
  cache-pixi-lock:
    runs-on: ubuntu-latest
    outputs:
      cache-key: ${{ steps.pixi-lock.outputs.cache-key }}
      fallback-key: ${{ steps.pixi-lock.outputs.fallback-key }}
      pixi-version: ${{ steps.pixi-lock.outputs.pixi-version }}
    steps:
      - uses: actions/checkout@v4
      - uses: Parcels-code/pixi-lock/create-and-cache@v1
        id: pixi-lock
        with:
          pixi-version: v0.63.0

  ci:
    needs: cache-pixi-lock
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: Parcels-code/pixi-lock/restore@v1
        with:
          cache-key: ${{ needs.cache-pixi-lock.outputs.cache-key }}
          fallback-key: ${{ needs.cache-pixi-lock.outputs.fallback-key }}
      - uses: prefix-dev/setup-pixi@v0.9.3
        with:
          pixi-version: ${{ needs.cache-pixi-lock.outputs.pixi-version }}
      # ... your CI steps
```

### Inputs & Outputs

#### `create-and-cache`

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `pixi-version` | Version of pixi to use for generating the lock file | No | `latest` |

| Output | Description |
|--------|-------------|
| `pixi-version` | The pixi version used |
| `cache-key` | The cache key (includes today's date) |
| `fallback-key` | The fallback cache key (yesterday's date) |

#### `restore`

| Input | Description | Required |
|-------|-------------|----------|
| `cache-key` | The cache key from `create-and-cache` | Yes |
| `fallback-key` | The fallback cache key from `create-and-cache` | Yes |

> [!NOTE]
> The cache key includes the current date, so the lock file is regenerated daily.
> The fallback key handles edge cases where the restore job runs just after midnight.

## Why not commit the lock file?

Committing your lock file is considered good practice when working on application code. Providing fixed package versions results in perfect reproducibility of environments between machines, and hence also reproducibility of results.

When developing and testing _library_ code, we don't want our environments to stay completely fixed - we want to test against environments covering a wide range of package versions to ensure compatability, including an environment includ the latest available versions of packages.

The easiest way to test against the latest versions of packages - and avoid the noisy commit history (and additional overhead) of regularly updating a lock file in git - is instead to ignore the lock file and rely on developers and CI to generate their own lock files. This forgoes perfect reprodubility between developer machines, and with CI machines.


## Dev notes

### Release checklist

- Update the `README.md` bumping the version of action
- Cut release
