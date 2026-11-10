# Example Recipe: Rust

```yaml
context:
  version: "0.1.0"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://github.com/example-package/example-package/archive/refs/tags/v${{ version }}.tar.gz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

build:
  number: 0
  script:
    env:
      CARGO_PROFILE_RELEASE_STRIP: symbols
      CARGO_PROFILE_RELEASE_LTO: fat
    content:
      - if: unix
        then:
          - cargo auditable install --locked --no-track --bins --root ${{ PREFIX }} --path .
        else:
          - cargo auditable install --locked --no-track --bins --root %LIBRARY_PREFIX% --path .
      - cargo-bundle-licenses --format yaml --output ./THIRDPARTY.yml

requirements:
  build:
    - ${{ stdlib('c') }}
    - ${{ compiler('c') }}
    - ${{ compiler('rust') }}
    - cargo-bundle-licenses
    - cargo-auditable

tests:
  - script: example-package --help
  - package_contents:
      bin:
        - example-package
      strict: true

about:
  homepage: https://github.com/example-package/example-package
  summary: Summary of the package
  description: |
    Description of the package
  license: MIT OR Apache-2.0
  license_file:
    - LICENSE-APACHE
    - LICENSE-MIT
    - THIRDPARTY.yml
  documentation: https://docs.rs/example-package
  repository: https://github.com/example-package/example-package

extra:
  recipe-maintainers:
    - your-github-username
```

Key points:
- Use `cargo-auditable` for auditable builds
- Use `cargo-bundle-licenses` to collect dependency licenses into THIRDPARTY.yml
- Strip symbols and enable LTO for smaller binaries
- Install to `${{ PREFIX }}` on Unix, `%LIBRARY_PREFIX%` on Windows
- If you are creating a CLI package that supports shell completions, you might want to suggest to the user that the recipe can include them as well. See [Shell Completions for CLI Packages](shell-completions.md).
- For Python packages with a Rust extension built by maturin, see the abi3 section in [example-recipe-python.md](example-recipe-python.md) instead.

## Frequent Fixes

Workarounds that come up repeatedly across Rust feedstocks. Apply them only when you hit the matching error; they are not part of the default template.

### linux-ppc64le: switch from rust-lld to GNU ld

**Symptom** (cross-compiled `linux_ppc64le: linux_64`): link fails with `unknown relocation (31) against symbol ...` or errors mentioning `R_PPC64_PLTSEQ` / `R_PPC64_PLTCALL` / `R_PPC64_PLT16_HA`. rustc links through the bundled rust-lld, which can't handle the inline-PLT relocations that the ppc64le gcc emits into the objects of C-based crates (`ring`, `aws-lc-sys`, `zstd-sys`, `libdbus-sys`, `onig_sys`, jemalloc, vendored libsolv, ...).

**Fix**: append `-fuse-ld=bfd`. gcc honours the last `-fuse-ld`, so this overrides rustc's lld choice:

```yaml
build:
  script:
    content:
      - if: target_platform == "linux-ppc64le"
        then:
          - export CARGO_BUILD_RUSTFLAGS="${CARGO_BUILD_RUSTFLAGS:-} -C link-arg=-fuse-ld=bfd"
      - cargo auditable install ...
```

- Append to `CARGO_BUILD_RUSTFLAGS` and don't set `RUSTFLAGS`. The rust activation puts the `$PREFIX` rpath flags in `CARGO_BUILD_RUSTFLAGS`, and cargo ignores `CARGO_BUILD_RUSTFLAGS` entirely once `RUSTFLAGS` is set. If you have to use `RUSTFLAGS`, seed it from the activation value: `export RUSTFLAGS="${RUSTFLAGS:-${CARGO_BUILD_RUSTFLAGS:-}} -C link-arg=-fuse-ld=bfd"`.
- `-fuse-ld=bfd` is the only stable way to switch on this target. The documented opt-out `-C linker-features=-lld` is still unstable on `powerpc64le-unknown-linux-gnu`. `-C link-self-contained=no`, which some recipes add as well, doesn't help on its own: rustc keeps passing `-fuse-ld=lld` and only drops the path to its bundled lld.

### Cross-compiling: target CFLAGS leak into build-script compiles

**Symptom** (any `build_platform != target_platform` Linux build): a C crate compiled for the *build* machine (e.g. `zstd-sys` pulled in through a build dependency) fails because the x86_64 compiler gets target flags like `-mcpu=power8`, `-mabi=lp64d` or `-march=...`. cc-rs combines the generic `$CFLAGS` with the target-scoped flags for every compile.

**Fix**: the rust activation already exports per-triple `CFLAGS_<triple>` / `CXXFLAGS_<triple>` / `CPPFLAGS_<triple>`, so drop the generic ones:

```yaml
      - if: linux and build_platform != target_platform
        then:
          - unset CFLAGS CXXFLAGS CPPFLAGS
```

Alternatively, move them to `TARGET_CFLAGS` / `TARGET_CXXFLAGS` (which cc-rs applies to the target only) before unsetting.

### aws-lc-sys

- **jitterentropy `-O0` failure** (all platforms): conda's `CFLAGS` inject `-O2`, which overrides the `-O0` that `jitterentropy-base.c` requires. Set `AWS_LC_SYS_CMAKE_BUILDER: "1"` in `build.script.env` and add `cmake` to the build requirements.
- **linux-ppc64le cross with the CMake builder**: `aes_hw_*` / `gcm_*_p8` symbols are undefined because CMake doesn't assemble `aesp8-ppc.S` / `ghashp8-ppc.S` when cross-compiling. On ppc64le, `unset AWS_LC_SYS_CMAKE_BUILDER` to use the cc builder; jitterentropy isn't part of the ppc64le sources.
- **win-arm64** (`LNK1181` on `chacha-armv8.o`, or `D9024`/`D9027` warnings): `cl.exe` can't assemble aws-lc's GAS-syntax ARM64 `.S` files. aws-lc-sys picks clang-cl on its own only when no `CC` is set, and the conda-forge activation sets one. Add `clang` to the build requirements and point only this crate at clang-cl:

  ```yaml
      - if: target_platform == "win-arm64"
        then:
          - set "AWS_LC_SYS_CC=clang-cl"
  ```

  If `AWS_LC_SYS_CC` alone doesn't work, use the CMake builder instead: set `AWS_LC_SYS_CMAKE_BUILDER=1` and `CC_aarch64_pc_windows_msvc=clang-cl.exe` / `CXX_aarch64_pc_windows_msvc=clang-cl.exe`, add `cmake` and `ninja` to the build requirements, and shorten paths with `CARGO_TARGET_DIR=%TMP%\<pkg>-target` so CMake's object paths stay under the Windows path limit.
- **win-64 → win-arm64 cross** (`LNK1112: module machine type 'x64' conflicts with target machine type 'ARM64'`): the default builder emits x86_64 objects; `AWS_LC_SYS_CMAKE_BUILDER=1` fixes it.
- `ring` (e.g. pulled in by rustls) also needs clang on win-arm64. If the dependency sits behind an optional feature, disabling that feature on win-arm64 is often the simpler fix.

### jemalloc on aarch64 / ppc64le

Crates using `jemalloc-sys` / `tikv-jemallocator` detect the page size of the build machine. On 64K-page aarch64/ppc64le systems the binary then crashes with `<jemalloc>: Unsupported system page size`. Build with a 64 KiB page size (the value is log2):

```yaml
build:
  script:
    env:
      JEMALLOC_SYS_WITH_LG_PAGE: ${{ '16' if (aarch64 or ppc64le) else '' }}
```

### Link against conda-forge libraries instead of vendored copies

- OpenSSL (`openssl-sys`): add `openssl` to host and `export OPENSSL_DIR=$PREFIX` on unix, which also makes cross-compiles find the target OpenSSL.
- Other `-sys` crates have similar switches, e.g. `LIBSQLITE3_SYS_USE_PKG_CONFIG=1` (with `libsqlite` in host), `ZSTD_SYS_USE_PKG_CONFIG=1` (with `zstd` in host), `ORT_PREFER_DYNAMIC_LINK=1` (with `onnxruntime`). When cross-compiling through pkg-config, also set `PKG_CONFIG_ALLOW_CROSS=1`.
- `prost-build` needs `protoc`: add `libprotobuf` to build and set `PROTOC=$BUILD_PREFIX/bin/protoc` (plus `PROTOC_NO_VENDOR=1`) so it doesn't build its vendored protobuf with CMake.

### Out of memory or disk while compiling or linking

Large workspaces can get the compiler or linker killed (`signal: 9, SIGKILL`, `terminated by a deadly signal`) on CI runners. Try these in order:

1. Limit parallelism with `export CARGO_BUILD_JOBS=2`.
2. Relax LTO: `CARGO_PROFILE_RELEASE_LTO: thin` or `"off"`, and/or `CARGO_PROFILE_RELEASE_CODEGEN_UNITS: "16"`.
3. For disk or memory pressure on Linux runners, add this to `conda-forge.yml` and rerender:

   ```yaml
   workflow_settings:
     free_disk_space: quick
     pagefile_size:
       - os: linux
         value: 15
   ```

### When a fix isn't worth it

linux-ppc64le is a low-priority platform. If a dependency fundamentally can't be built there, it's acceptable to drop the platform (remove `linux_ppc64le` from `conda-forge.yml` and rerender, or add `skip: ppc64le`), or to build with fewer optional cargo features on ppc64le only. Mention this to the user rather than doing it silently.
