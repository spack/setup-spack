# Setup Spack in Github Actions

Set up the [Spack package manager](https://github.com/spack/spack).

## Example 1 (`spack install`)

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
        concretizer:
        - original
        - clingo

    steps:
      - uses: actions/checkout@v1.0.0
      - name: Set up Spack
        uses: haampie-spack/setup-spack@v1.2.0
        with:
          os: ${{ matrix.os }}
          concretizer: ${{ matrix.concretizer }}
          ref: develop
      - run: |
        spack --version
        spack install zlib
```

## Example 2 (caching binaries)

Assuming you have an environment file in `example-environment/spack.yaml`, you
can enable caching by using the manifest file `spack.lock` as a hash key.

```yaml
jobs:
  latest-stable:
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
        uses: haampie-spack/setup-spack@v1.2.0
        with:
          os: ${{ matrix.os }}
          ref: develop
          concretizer: clingo

      - name: Concretize environment
        run: |
          spack --color always -e ./example-environment concretize -f

      - name: Restore Spack cache
        uses: actions/cache@v2
        with:
          path: ~/.spack-ci
          key: ${{ matrix.os }}-spack-${{ hashFiles('**/spack.lock') }}
          restore-keys: |
            ${{ matrix.os }}-spack

      - name: Install environment
        run: |
          spack --color always -e ./example-environment install -j3 -v
```

## How is Spack bootstrapped?

This environment is built

https://github.com/haampie-spack/setup-spack/blob/rebuild-spack/spack.yaml

and the binaries are uploaded as release assets to

https://github.com/haampie-spack/setup-spack/releases/tag/develop

Todo:
- [ ] Add checksum verification
