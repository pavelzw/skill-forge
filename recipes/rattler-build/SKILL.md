---
name: rattler-build
description: >-
  Build conda packages with rattler-build: create new recipes, build and debug
  them, create patches, and inspect built packages. Use when working with
  rattler-build recipes, conda package building, or when the user mentions
  rattler-build.
license: BSD-3-Clause
compatibility: >-
  Requires rattler-build, curl, jq, and standard Unix tools; some workflows also
  use unzip, zstd (or tar with --zstd), and platform-specific tools like
  otool/readelf and patchelf.
---

# rattler-build Operations

Determine which workflow applies based on the task:

| Task | Workflow |
|------|----------|
| Create a new recipe for a package | [references/new-recipe.md](references/new-recipe.md) |
| Build an existing recipe | [references/build-recipe.md](references/build-recipe.md) |
| Debug a failed build | [references/debug-build.md](references/debug-build.md) |
| Create a patch from build dir changes | [references/create-patch.md](references/create-patch.md) |
| Inspect a built `.conda` package | [references/inspect-package.md](references/inspect-package.md) |

---

## Recipe Format Reference

A rattler-build recipe (`recipe.yaml`) has these key sections:

```yaml
context:
  name: mypackage
  version: "1.0.0"

package:
  name: ${{ name }}
  version: ${{ version }}

source:
  url: https://example.com/source-${{ version }}.tar.gz
  sha256: <hash>

build:
  number: 0
  script:
    - cmake -B build -G Ninja -DCMAKE_INSTALL_PREFIX=${{ PREFIX }}
    - cmake --build build
    - cmake --install build

requirements:
  build:
    - ${{ compiler('c') }}
    - ${{ compiler('cxx') }}
    - cmake
    - ninja
  host:
    - zlib
  run:
    - zlib

tests:
  - package_contents:
      bin:
        - myprogram
  - script:
      - myprogram --version

about:
  homepage: <url>
  license: <SPDX-license>
  license_file: LICENSE
  summary: <one-line summary>
```

### Multi-output recipe

A single recipe can produce multiple packages using `outputs`. When using outputs,
replace the top-level `package` with `recipe` and define each output separately:

```yaml
context:
  version: "1.0.0"

recipe:
  name: mylib-split
  version: ${{ version }}

source:
  url: https://example.com/mylib-${{ version }}.tar.gz
  sha256: <hash>

build:
  number: 0

outputs:
  - package:
      name: libmylib
    requirements:
      build:
        - ${{ compiler('c') }}
        - cmake
        - ninja
      host:
        - zlib
      run:
        - zlib
    build:
      script:
        - cmake -B build -G Ninja -DCMAKE_INSTALL_PREFIX=${{ PREFIX }}
        - cmake --build build
        - cmake --install build
    tests:
      - package_contents:
          lib:
            - mylib

  - package:
      name: mylib-dev
    requirements:
      run:
        - ${{ pin_subpackage('libmylib', exact=True) }}
    build:
      script:
        - mkdir -p $PREFIX/include
        - cp -R include/* $PREFIX/include/
    tests:
      - package_contents:
          include:
            - mylib.h
```

Each output acts as an independent recipe with its own `build`, `requirements`, and
`tests`. Use `pin_subpackage()` to create dependencies between outputs.

### Virtual packages

Virtual packages represent system properties that the package manager cannot install
but can use for dependency resolution. They are prefixed with double underscores and
injected by the solver based on the target system. Common virtual packages:

- `__linux` - present on Linux, version matches kernel version
- `__osx` - present on macOS, version matches macOS version
- `__glibc` - glibc version on the system (e.g., `__glibc >=2.17`)
- `__cuda` - CUDA driver version if available
- `__archspec` - CPU microarchitecture (e.g., `x86_64_v3`)

Use them in `run_constraints` to declare system-level compatibility requirements
without pulling in an actual dependency:

```yaml
requirements:
  run_constraints:
    - __glibc >=2.17
```

### Key concepts

- **`build` requirements**: tools that run during the build (compilers, cmake, make, ninja)
- **`host` requirements**: libraries linked against (`$PREFIX`)
- **`run` requirements**: runtime dependencies
- **`run_constraints`**: optional runtime deps that must satisfy the version spec if installed
- Use `${{ compiler('c') }}` for C compiler, `${{ compiler('cxx') }}` for C++
- `$PREFIX` is the host prefix where things get installed (use in build scripts)
- `${{ PREFIX }}` is the Jinja expression for use inside YAML fields
- `$BUILD_PREFIX` is where build tools live
- Use `${{ CPU_COUNT }}` for parallel make jobs
- Use SPDX license identifiers
- Set `build.number` to 0 for new recipes
