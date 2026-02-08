# Setup Spack in GitHub Actions

Set up the [Spack package manager](https://github.com/spack/spack) with a default build cache to
speed up your actions.

## Options

| Name                  | Description                                      | Default                 |
|-----------------------|--------------------------------------------------|-------------------------|
| `spack_ref`           | Version of Spack tool                            | `"develop"`             |
| `packages_ref`        | Version of Spack packages                        | `"develop"`             |
| `buildcache`          | Enable the GitHub Action build cache             | `true`                  |
| `color`               | Force color output                               | `true`                  |
| `spack_path`          | Path to install Spack to                         | `"./spack"`             |
| `packages_path`       | Path to install Spack packages repository        | `"./spack-packages"`    |
| `spack_repository`    | GitHub repository for Spack                      | `"spack/spack"`         |
| `packages_repository` | GitHub repository for Spack packages             | `"spack/spack-packages"`|

## How It Works

Spack v1.x uses separate repositories: `spack/spack` (the core) and `spack/spack-packages` (the package repository). This action automatically clones both repositories and configures them using `spack repo set --destination <clone dir> builtin`.

**Version independence**: The `spack_ref` and `packages_ref` are independently versioned. The core Spack tool follows a release cycle (e.g., `releases/v1.0`), while the packages repository has its own versioning (e.g., `v2025.11.0`). Use different refs when you need a specific package version with a specific Spack version.

## Example: basic setup

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
    - name: Set up Spack
      uses: spack/setup-spack@v3
      with:
        buildcache: true
    - run: spack install python
```

## Example: specific versions

To use specific versions of Spack and/or packages:

```yaml
    - name: Set up Spack
      uses: spack/setup-spack@v3
      with:
        spack_ref: releases/v1.0
        packages_ref: v2025.11.0
        buildcache: true
    - run: spack install python
```

## Example: using a fork

To test changes in your own Spack or packages fork:

```yaml
    - name: Set up Spack
      uses: spack/setup-spack@v3
      with:
        spack_repository: myorg/spack
        spack_ref: my-feature-branch
        packages_repository: spack/spack-packages  # Use official packages
        packages_ref: develop
        buildcache: true
    - run: spack install python
```

When `buildcache: true` is set, binaries from https://github.com/spack/github-actions-buildcache
are used. For available software, [see here](https://github.com/spack/github-actions-buildcache/blob/main/spack.yaml).

## Example: shell support

If you want to use shell-aware commands such as `spack env activate` and `spack load`,
use either `shell: spack-bash {0}` or `shell: spack-sh {0}` in your action:

```yaml
- name: Shell example
  shell: spack-bash {0}
  run: |
    spack env activate .
    spack env status
```

These "shells" are small wrappers that run `. setup-env.sh` before executing your script.

## Example: caching your own binaries for public repositories

When you need to install packages not available in the default build cache, you can build them
once and then cache them on GitHub Packages.

The easiest way to do so is to create a `spack.yaml` environment in the root of your git
repository:

```yaml
spack:
  view: /opt/view
  specs:
  - python@3.11

  config:
    install_tree:
      root: /opt/spack
      padded_length: 128

  packages:
    all:
      require: 'target=x86_64_v3'

  mirrors:
    local-buildcache:
      url: oci://ghcr.io/<username>/spack-buildcache
      binary: true
      signed: false
      access_pair:
        id_variable: GITHUB_USER
        secret_variable: GITHUB_TOKEN
```

Then configure an action like this:

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    permissions:
      packages: write
    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Set up Spack
      uses: spack/setup-spack@v3

    - name: Install
      run: spack -e . install

    - name: Run
      shell: spack-bash {0}
      run: |
        spack env activate .
        python3 -c 'print("hello world")'

    - name: Push packages and update index
      env:
        GITHUB_USER: ${{ github.actor }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: spack -e . buildcache push --base-image ubuntu:22.04 --update-index local-buildcache
      if: ${{ !cancelled() }}
```

## Example: caching your own binaries for private repositories

When your local buildcache is stored in a private GitHub package,
you need to specify the OCI credentials already *before* `spack concretize`.
This is because Spack needs to fetch the buildcache index.

```yaml
env:
  GITHUB_USER: ${{ github.actor }}
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

jobs:
  example-private:
    steps:
    - name: Set up Spack
      uses: spack/setup-spack@v3

    - name: Concretize
      run: spack -e . concretize

    - name: Install
      run: spack -e . install

    - name: Push packages and update index
      run: spack -e . buildcache push --base-image ubuntu:22.04 --update-index local-buildcache
```

From a security perspective, do note that the `GITHUB_TOKEN` is exposed to every
job step.

## License

This project is part of Spack. Spack is distributed under the terms of both the
MIT license and the Apache License (Version 2.0). Users may choose either
license, at their option.

All new contributions must be made under both the MIT and Apache-2.0 licenses.

See LICENSE-MIT, LICENSE-APACHE, COPYRIGHT, and NOTICE for details.

SPDX-License-Identifier: (Apache-2.0 OR MIT)

LLNL-CODE-811652
