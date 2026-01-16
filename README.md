# pixi-lock

> [!NOTE]
> This repo is likely to be moved to https://github.com/xarray-contrib

This repo provides two GitHub Actions for managing `pixi.lock` files with caching:

- **`create-and-cache`**: Generates a `pixi.lock` file and caches it
- **`restore`**: Restores the cached `pixi.lock` file in downstream jobs

This two-action pattern is designed for CI workflows where you want to generate the lock file once and reuse it across multiple matrix jobs.


## Usage

```yaml
env:
  PIXI_VERSION: "v0.63.0"

jobs:
  cache-pixi-lock:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: Parcels-code/pixi-lock/create-and-cache@v1
        with:
          pixi-version: ${{ env.PIXI_VERSION }}

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
          pixi-version: ${{ env.PIXI_VERSION }}
      - uses: prefix-dev/setup-pixi@v0.9.3
        with:
          pixi-version: ${{ env.PIXI_VERSION }}
      # ... your CI steps
```

> [!NOTE]
> Using the same Pixi version in CI as from lockfile generation ensures better
> stability. Otherwise updates to Pixi which introduce breaking changes in:
> - (a) the format of the lock file - which would break your CI for the day, or
> - (b) the format of `pixi.toml` - which would break your CI until you fix it.

## Why not commit the lock file?

Committing your lock file is considered good practice when working on application code. Providing fixed package versions results in perfect reproducibility of environments between machines, and hence also reproducibility of results.

When developing and testing _library_ code, we don't want our environments to stay completely fixed - we want to test against environments covering a wide range of package versions to ensure compatability, including an environment includ the latest available versions of packages.

The easiest way to test against the latest versions of packages - and avoid the noisy commit history (and additional overhead) of regularly updating a lock file in git - is instead to ignore the lock file and rely on developers and CI to generate their own lock files. This forgoes perfect reprodubility between developer machines, and with CI machines.


## Dev notes

### Release checklist

- Update the `README.md` bumping the version of action
- Cut release
