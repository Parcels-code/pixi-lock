# create-pixi-lock

> [!NOTE]
> This repo is likely to be moved to https://github.com/xarray-contrib

This action creates a `pixi.lock` file and caches it for future use. Subsequent runs of this action on the same day restore the lock file instead of generating it anew.

## Why not commit the lock file?

Committing your lock file is considered good practice when working on application code. Providing fixed package versions results in perfect reproducibility of environments between machines, and hence also reproducibility of results.

When developing and testing _library_ code, we don't want our environments to stay completely fixed - we want to test against environments covering a wide range of package versions to ensure compatability, including an environment includ the latest available versions of packages.

The easiest way to test against the latest versions of packages - and avoid the noisy commit history (and additional overhead) of regularly updating a lock file in git - is instead to ignore the lock file and rely on developers and CI to generate their own lock files. This forgoes perfect reprodubility between developer machines, and with CI machines.
