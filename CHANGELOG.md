# Changelog

## v3.0.0

- **BREAKING**: Renamed `ref` input to `spack_ref` for clarity
- **BREAKING**: Renamed `repository` input to `spack_repository` for consistency
- **BREAKING**: Renamed `path` input to `spack_path` for consistency
- Support Spack v1.x with separate `spack/spack` and `spack/spack-packages` repositories
- Added new inputs: `packages_ref`, `packages_repository`, and `packages_path`
- Automatically clones and configures the packages repository using `spack repo set`
- Removed support for Spack v0.x

## v2.1.1

- For Spack `v0.22.0.dev0` and higher `spack install` without further flags can
  install prebuilt binaries from the build cache without erroring about missing
  signatures on the binaries. The default build cache is marked as "unsigned".
- Disable `/etc/spack` and `~/.spack` config files, only use `$spack/etc`

## v2.1.0

- Support shell aware commands like `spack env activate` and `spack load` in the `spack-bash`
  and `spack-sh` shells.

## v2.0.0

- Enable build cache from https://github.com/spack/github-actions-buildcache
- Enable color output by default
- Remove custom bootstrapped binaries for clingo
- Remove option for choosing the concretizer
