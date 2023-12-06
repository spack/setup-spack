# Setup Spack in Github Actions

Set up the [Spack package manager](https://github.com/spack/spack) with a default build cache to
speed up your actions.

## Example

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
    - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
    - name: Set up Spack
      uses: spack/setup-spack@v2
      with:
        ref: develop      # Spack version (examples: develop, releases/v0.21)
        buildcache: true  # Configure oci://ghcr.io/spack/github-actions-buildcache
        color: true       # Force color output (SPACK_COLOR=always)
        path: spack       # Where to clone Spack
    - run: spack install --no-check-signature --reuse python
```

## Using prebuilt binaries

When `buildcache: true` is set, binaries from https://github.com/spack/github-actions-buildcache
are reused.

These binaries are unsigned, so you have to specify `install --no-check-signature`.

## Shell support

This action does not enable shell support, so you can't use `spack load` directly.

Consider using environment views instead.

## Using your own build cache

See https://github.com/spack/github-actions-buildcache for details on how to cache binaries you've
built yourself.

## License

This project is part of Spack. Spack is distributed under the terms of both the
MIT license and the Apache License (Version 2.0). Users may choose either
license, at their option.

All new contributions must be made under both the MIT and Apache-2.0 licenses.

See LICENSE-MIT, LICENSE-APACHE, COPYRIGHT, and NOTICE for details.

SPDX-License-Identifier: (Apache-2.0 OR MIT)

LLNL-CODE-811652