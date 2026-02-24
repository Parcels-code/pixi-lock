# pixi-lock

> [!NOTE]
> This repo may be moved to https://github.com/prefix-dev

This repo provides two GitHub Actions for generating and caching `pixi.lock` files in CI:

- **`create-and-cache`**: Generates a `pixi.lock` file and caches it
- **`restore`**: Restores the cached `pixi.lock` file in downstream jobs

This two-action pattern is so that the lockfile can be omitted from the git
history, but still be generated in a performant manner (i.e., regenerated
and cached with a key that depends on `pixi.toml` and the date -
then shared across jobs).

## Usage

```yaml
jobs:
  cache-pixi-lock:
    runs-on: ubuntu-slim
    outputs:
      cache-key: ${{ steps.pixi-lock.outputs.cache-key }}
      pixi-version: ${{ steps.pixi-lock.outputs.pixi-version }}
    steps:
      - uses: actions/checkout@v4
      - uses: Parcels-code/pixi-lock/create-and-cache@... # TODO: Copy the hash for the rev you want to install from
        id: pixi-lock
        with:
          pixi-version: ... # TODO: update with your selected pixi version
      - uses: actions/upload-artifact@v6 # make available as an artifact for local testing
        with:
          name: pixi-lock
          path: pixi.lock

  ci:
    needs: cache-pixi-lock
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: Parcels-code/pixi-lock/restore@ # TODO: Copy the hash for the rev you want to install from (same as above)
        with:
          cache-key: ${{ needs.cache-pixi-lock.outputs.cache-key }}
      - uses: prefix-dev/setup-pixi@v... # TODO: update with your selected setup-pixi version
        with:
          pixi-version: ${{ needs.cache-pixi-lock.outputs.pixi-version }}
      # ... your CI steps
```

### Inputs & Outputs

#### `create-and-cache`

| Input          | Description                                             | Required | Default                          |
| -------------- | ------------------------------------------------------- | -------- | -------------------------------- |
| `pixi-version` | Version of pixi to use for generating the lock file     | No       | `latest`                         |
| `hash-files`   | Files to use to generate the hash key for the lock file | No       | `pixi.toml` and `pyproject.toml` |

| Output         | Description                           |
| -------------- | ------------------------------------- |
| `pixi-version` | The pixi version used                 |
| `cache-key`    | The cache key (includes today's date) |

#### `restore`

| Input       | Description                           | Required |
| ----------- | ------------------------------------- | -------- |
| `cache-key` | The cache key from `create-and-cache` | Yes      |

> [!NOTE]
> The cache key includes the current date, so the lock file is regenerated daily.
> The fallback key (yesterday's date) is calculated automatically to handle edge cases where the restore job runs just after midnight.

## Why not commit the lock file?

Committing your lock file is considered good practice when working on application code. Providing fixed package versions results in perfect reproducibility of environments between machines, and hence also reproducibility of results.

When developing and testing _library_ code, we don't want our environments to stay completely fixed - we want to test against environments covering a wide range of package versions to ensure compatability, including an environment includ the latest available versions of packages.

The _easiest_ way to test against the latest versions of packages - and avoid the noisy commit history (and additional overhead) of regularly updating a lock file in git - is instead to ignore the lock file and rely on developers and CI to generate their own lock files. This much simpler setup forgoes perfect reprodubility between developer machines, and with CI machines - which may be a worthwhile tradeoff for your project.

See the following threads for more detailed discussion:

- [prefix.dev Discord: Should you commit the lockfile](https://discord.com/channels/1082332781146800168/1462778624212996209)
- [Scientific Python Discord: lock files for libraries](https://discord.com/channels/786703927705862175/1450619697224487083)
- https://github.com/prefix-dev/pixi/issues/5325
