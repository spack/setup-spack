# Setup Spack in Github Actions

Set up Spack with the new concretizer enabled by default.

Example usage:

```yaml
jobs:
  build:
    runs-on: ${{ matrix.os }}

    strategy:
      fail-fast: false
      matrix:
        os:
        - ubuntu-18.04
        - ubuntu-20.04
        - macos-10.15

    steps:
      - uses: actions/checkout@v1.0.0
      - name: Set up Spack
        uses: haampie-spack/setup-spack@v1
        with:
          os: ${{ matrix.os }}
          ref: develop
      - run: |
        spack --version
        spack install zlib
```



## How is Spack bootstrapped?

This environment is built

https://github.com/haampie-spack/setup-spack/blob/rebuild-spack/spack.yaml

and the binaries are uploaded as release assets to

https://github.com/haampie-spack/setup-spack/releases/tag/develop

Todo:
- [ ] Add checksum verification