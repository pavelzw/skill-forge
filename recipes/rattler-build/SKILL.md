---
name: rattler-build
description: >-
  Build conda packages with rattler-build: create new recipes, build and debug
  them, create patches, and inspect built packages. Use when working with
  rattler-build recipes, conda package building, or when the user mentions
  rattler-build.
license: BSD-3-Clause
compatibility: >-
  Requires rattler-build, curl, jq, and standard Unix tools.
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

### Key concepts

- **`build` requirements**: tools that run during the build (compilers, cmake, make, ninja)
- **`host` requirements**: libraries linked against (`$PREFIX`)
- **`run` requirements**: runtime dependencies
- Use `${{ compiler('c') }}` for C compiler, `${{ compiler('cxx') }}` for C++
- `$PREFIX` is the host prefix where things get installed
- `$BUILD_PREFIX` is where build tools live
- Use `${{ CPU_COUNT }}` for parallel make jobs
- Use SPDX license identifiers
- Set `build.number` to 0 for new recipes
