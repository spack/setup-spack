# Setup Spack in GitHub Actions

Set up the [Spack package manager](https://github.com/spack/spack) with a default build cache to
speed up your actions.

## Options

| Name        | Description                          | Default        |
|-------------|--------------------------------------|----------------|
| `ref`       | Version of Spack                     | `"develop"`    |
| `buildcache`| Enable the GitHub Action build cache | `true`         |
| `color`     | Force color output                   | `true`         |
| `path`      | Path to install Spack to             | `"spack"`      |
| `repository`| GitHub repository for Spack          | `"spack/spack"`|

## Example: basic setup

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
    - name: Set up Spack
      uses: spack/setup-spack@v2
      with:
        ref: develop      # Spack version (examples: develop, releases/v0.23)
        buildcache: true  # Configure oci://ghcr.io/spack/github-actions-buildcache
        color: true       # Force color output (SPACK_COLOR=always)
        path: spack       # Where to clone Spack
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
      uses: spack/setup-spack@v2

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
