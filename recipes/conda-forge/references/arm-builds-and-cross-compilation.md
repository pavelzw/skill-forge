# ARM Builds and Cross-Compilation

## Adding Native ARM Builds to a Feedstock

To enable the standard ARM builds on conda-forge, add the following to `conda-forge.yml`:

```yaml
provider:
  osx_arm64: default
  linux_aarch64: default
```

Then rerender:
```bash
pixi exec conda-smithy rerender --commit=all
```

This generates new CI variant configs in `.ci_support/` for the additional platforms. These are native builds, not cross-compilation: `osx_arm64` currently defaults to Azure, while `linux_aarch64` currently defaults to GitHub Actions.

## What the settings mean

| Key | Value | Effect |
|-----|-------|--------|
| `provider.osx_arm64` | `default` | Build Apple Silicon packages natively on Azure |
| `provider.linux_aarch64` | `default` | Build ARM64 Linux packages natively on GitHub Actions |

## Platform types

- **Build platform**: Where the compiler/toolchain runs (e.g., `linux_64`)
- **Host platform**: Target architecture for the built binaries (e.g., `linux_aarch64`)
- **Target platform**: Only relevant for cross-compilers; usually same as host

## linux-ppc64le

`linux-ppc64le` is discouraged for new enablement. Do not add it by default together with `linux_aarch64`.

Only opt in when there is a concrete requirement for `linux-ppc64le` and you are prepared for a slower, less common CI path.

## win-arm64

`win-arm64` is experimental.

Only enable it if the user explicitly asks for `win-arm64` support. Do not add it as part of the default ARM enablement path.

```yaml
build_platform:
  win_arm64: win_64
```

## Explicit cross-compilation

Only use `build_platform` when you intentionally need cross-compilation and the recipe is already prepared for it.

```yaml
build_platform:
  <target_platform>: <build_platform>
test: native_and_emulated
```

## Recipe requirements for explicit cross-compilation

### Build vs Host dependencies

**Build** (tools that run during build): compilers, cmake, make, ninja, pkg-config

**Host** (files for the target platform): libraries, headers, python, numpy

### Python cross-compilation

Add conditional cross-compilation dependencies:

```yaml
requirements:
  build:
    - ${{ compiler('c') }}
    - ${{ stdlib('c') }}
    - if: build_platform != target_platform
      then:
        - cross-python_${{ target_platform }}
        - python
        - cython # only if used in host
        - numpy # only if used in host
        - maturin # only if used in host
  host:
    - python
    - cython
    - numpy
    - maturin
```

### Testing cross-compiled builds

Native `provider`-based `osx_arm64` and `linux_aarch64` builds run tests normally.

For explicit cross-builds, `test: native_and_emulated` allows testing only when the target can be run natively or via emulation. Cross-built `linux_aarch64` jobs can often be tested under emulation, while cross-built `osx_arm64` jobs from `osx_64` still cannot be emulated.
