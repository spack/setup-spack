# Setup Spack in Github Actions

Set up the [Spack package manager](https://github.com/spack/spack) with a default build cache to
speed up your actions.

## Example: basic setup

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
    - name: Set up Spack
      uses: spack/setup-spack@v2
      with:
        ref: develop      # Spack version (examples: develop, releases/v0.21)
        buildcache: true  # Configure oci://ghcr.io/spack/github-actions-buildcache
        color: true       # Force color output (SPACK_COLOR=always)
        path: spack       # Where to clone Spack
    - run: spack install --no-check-signature --reuse python
```

When `buildcache: true` is set, binaries from https://github.com/spack/github-actions-buildcache
are reused. For available software, [see here](https://github.com/spack/github-actions-buildcache/blob/main/spack.yaml).

These binaries are unsigned, so you have to specify `install --no-check-signature`.

## Example: caching your own binaries for public repositories

When you need to install packages not available in the default build cache, you can build them
once and then cache them on GitHub Packages.

The easiest way to do so is to create a `spack.yaml` environment in the root of your git
repository:

```yaml
spack:
  view: my_view
  specs:
  - python@3.11

  config:
    install_tree:
      root: /opt/spack
      padded_length: 128

  packages:
    all:
      require: 'target=x86_64_v2'

  mirrors:
    local-buildcache: oci://ghcr.io/<username>/spack-buildcache
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
      run: spack -e . install --no-check-signature

    - name: Run
        run: ./my_view/bin/python3 -c 'print("hello world")'

    - name: Push packages and update index
      run: |
        spack -e . mirror set --push --oci-username ${{ github.actor }} --oci-password "${{ secrets.GITHUB_TOKEN }}" local-buildcache
        spack -e . buildcache push --base-image ubuntu:22.04 --unsigned --update-index local-buildcache
      if: ${{ !cancelled() }}
```

## Example: caching your own binaries for private repositories

When your local build cache is stored in a private GitHub package,
you need to specify the OCI credentials already *before* `spack concretize`.
This is because Spack needs to fetch the index of the build cache. Also, remember to
remove the `--push` flag from `spack mirror set`, since fetching needs
credentials too:

```yaml
    steps:
    - name: Login
      run: spack -e . mirror set --oci-username ${{ github.actor }} --oci-password "${{ secrets.GITHUB_TOKEN }}" local-buildcache

    - name: Concretize
      run: spack -e . concretize

    - name: Install
      run: spack -e . install --no-check-signature

    - name: Push packages and update index
      run: spack -e . buildcache push --base-image ubuntu:22.04 --unsigned --update-index local-buildcache
      if: ${{ !cancelled() }}
```

From a security perspective, notice that the `GITHUB_TOKEN` is exposed to every
subsequent job step. (This is no different from `docker login`, which also likes
to store credentials in the home directory.)


## Shell support

This action currently does not enable shell support, so you can't use `spack load` directly.

Environment views are recommended instead.

## License

This project is part of Spack. Spack is distributed under the terms of both the
MIT license and the Apache License (Version 2.0). Users may choose either
license, at their option.

All new contributions must be made under both the MIT and Apache-2.0 licenses.

See LICENSE-MIT, LICENSE-APACHE, COPYRIGHT, and NOTICE for details.

SPDX-License-Identifier: (Apache-2.0 OR MIT)

LLNL-CODE-811652