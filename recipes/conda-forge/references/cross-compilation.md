# Cross-Compilation and ARM Support

## Adding ARM / Cross-Compilation to a Feedstock

To enable cross-compilation for ARM and other architectures, add the following to `conda-forge.yml`:

```yaml
build_platform:
  osx_arm64: osx_64
  linux_aarch64: linux_64
  linux_ppc64le: linux_64
test: native_and_emulated
```

Then rerender:
```bash
pixi exec conda-smithy rerender --commit=all
```

This generates new CI variant configs in `.ci_support/` for the additional platforms.

## What the settings mean

| Key | Value | Effect |
|-----|-------|--------|
| `build_platform.osx_arm64` | `osx_64` | Build Apple Silicon packages on Intel macOS |
| `build_platform.linux_aarch64` | `linux_64` | Build ARM64 Linux packages on x86_64 Linux |
| `build_platform.linux_ppc64le` | `linux_64` | Build PPC64LE Linux packages on x86_64 Linux |
| `test` | `native_and_emulated` | Run tests using emulation where possible |

## Platform types

- **Build platform**: Where the compiler/toolchain runs (e.g., `linux_64`)
- **Host platform**: Target architecture for the built binaries (e.g., `linux_aarch64`)
- **Target platform**: Only relevant for cross-compilers; usually same as host

## Recipe requirements for cross-compilation

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

### Skipping tests for cross-compiled builds

In `conda-forge.yml`, you define `tests: native_and_emulated`.
Builds for `linux_aarch64` and `linux_ppc64le` can run tests because they can be emulated using QEMU.
Cross-builds for `osx_arm64` from `osx_64` cannot run tests because they can't be emulated.

### linux-ppc64le

`linux-ppc64le` only works for Go recipes at the moment. Only include it when you are faced with a cross-compilation scenario where `${{ compiler('go-nocgo') }}` is used.
