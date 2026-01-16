# create-pixi-lock

> [!NOTE]
> This repo is likely to be moved to https://github.com/xarray-contrib

This action creates a `pixi.lock` file and caches it for future use. Subsequent runs of this action on the same day restore the lock file instead of generating it anew.


## Usage

Basic usage with a set version of Pixi as well as daily cached lockfile:

```yaml
env:
  PIXI_VERSION: "v0.63.0"

# ...
steps:
  - uses: actions/checkout@v4

  - uses: Parcels-code/create-pixi-lock@v1
    with:
      pixi-version: ${{ env.PIXI_VERSION }} # Default is latest - same as setup-pixi
      cache-frequency: daily # options are "daily", "weekly", "monthly". Default "daily"
  - uses: prefix-dev/setup-pixi@v0.9.3
    with:
      pixi-version: ${{ env.PIXI_VERSION }}
```

> [!NOTE]
> Pinning your Pixi version in CI and updating this pin manually ensures better
> stability. Otherwise updates to Pixi which introduce breaking changes in:
> - (a) the format of the lock file - which would break your CI for a period of `cache-frequency`, or
> - (b) the format of `pixi.toml` - which would break your CI until you fix it.

## Why not commit the lock file?

Committing your lock file is considered good practice when working on application code. Providing fixed package versions results in perfect reproducibility of environments between machines, and hence also reproducibility of results.

When developing and testing _library_ code, we don't want our environments to stay completely fixed - we want to test against environments covering a wide range of package versions to ensure compatability, including an environment that includes the latest available versions of packages.

The easiest way to test against the latest versions of packages - and avoid the noisy commit history (and additional overhead) of regularly updating a lock file in git - is instead to ignore the lock file and rely on developers and CI to generate their own lock files. This forgoes perfect reprodubility between developer machines, and with CI machines.


## Dev notes

### Release checklist

- Update the `README.md` bumping the version of action
- Cut release