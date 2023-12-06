# Setup Spack in Github Actions

Set up the [Spack package manager](https://github.com/spack/spack) with a default build cache to
speed up your actions.

## Example

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v1.0.0
      - name: Set up Spack
        uses: haampie-spack/setup-spack@v1.2.0
        with:
          ref: develop
          buildcache: true
      - run: |
        spack --version
        spack install gmake
```

## Caching your own binaries

See https://github.com/spack/github-actions-buildcache for details on how to cache binaries you've
built yourself.
