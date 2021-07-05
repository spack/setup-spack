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

    steps:
      - uses: actions/checkout@v1.0.0
      - name: "Set up Spack"
        uses: haampie-spack/setup-spack@v1
        with:
          os: ${{ matrix.os }}
      - run: spack --version
      - run: spack install zlib
```