# Changelog

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
