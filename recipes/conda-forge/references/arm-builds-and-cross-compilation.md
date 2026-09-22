# ARM Builds and Cross-Compilation

## Adding Native ARM Builds to a Feedstock

To enable the standard ARM builds on conda-forge, add the following to `conda-forge.yml`:

```yaml
provider:
  osx_arm64: default
  linux_aarch64: default
  win_arm64: default
build_platform:
  linux_riscv64: linux_64
```

Then rerender:
```bash
pixi exec conda-smithy rerender --commit=all
```

This generates new CI variant configs in `.ci_support/` for the additional platforms. These are native builds, not cross-compilation: `osx_arm64` currently defaults to Azure, while `linux_aarch64` and `win_arm64` currently default to GitHub Actions.
Since `linux_riscv64` currently doesn't have native runners, they should be built using cross-compilation.

## What the settings mean

| Key | Value | Effect |
|-----|-------|--------|
| `provider.osx_arm64` | `default` | Build Apple Silicon packages natively on Azure |
| `provider.linux_aarch64` | `default` | Build ARM64 Linux packages natively on GitHub Actions |
| `provider.win_arm64` | `default` | Build ARM64 Windows packages natively on GitHub Actions |
| `build_platform.linux_riscv64` | `linux_64` | Build RISCV64 Linux packages through cross-compilation on GitHub Actions |

## Platform types

- **Build platform**: Where the compiler/toolchain runs (e.g., `linux_64`)
- **Host platform**: Target architecture for the built binaries (e.g., `linux_aarch64`)
- **Target platform**: Only relevant for cross-compilers; usually same as host

## linux-ppc64le

`linux-ppc64le` is discouraged for new enablement. Do not add it by default together with `linux_aarch64`.

Only opt in when there is a concrete requirement for `linux-ppc64le` and you are prepared for a slower, less common CI path.

## Explicit cross-compilation

Only use `build_platform` when you intentionally need cross-compilation and the recipe is already prepared for it.

```yaml
build_platform:
  <target_platform>: <build_platform>
test: native_and_emulated
```

When building for `win-arm64` a package that requires `${{ compiler('go-cgo') }}`, you need explicit cross-compilation from `win-64` as there is no native `go-cgo` compiler for `win-arm64`.

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
