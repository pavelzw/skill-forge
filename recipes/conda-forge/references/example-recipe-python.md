# Example Recipe: Python

## Pure Python (noarch)

```yaml
context:
  version: "1.0.0"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://files.pythonhosted.org/packages/source/e/example-package/example_package-${{ version }}.tar.gz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

build:
  number: 0
  python:
    entry_points:
      - example-command=example_package.cli:cli
  noarch: python
  script: python -m pip install . -vv --no-deps --no-build-isolation

requirements:
  host:
    - python ${{ python_min }}.*
    - pip
    - hatchling
  run:
    - python >=${{ python_min }}
    - requests
    - click

tests:
  - python:
      imports:
        - example_package
      pip_check: true
      python_version:
        - ${{ python_min }}.*
        - "*"
  - script:
      - example-command --help

about:
  homepage: https://github.com/example-org/example-package
  summary: Short description of the package
  license: MIT
  license_file: LICENSE

extra:
  recipe-maintainers:
    - your-github-username
```

Key points:
- Use `noarch: python` for pure Python packages (no compiled extensions)
- Use `python_min` for version pinning: `${{ python_min }}.*` in host, `>=${{ python_min }}` in run, and `python_version: [${{ python_min }}.*, "*"]` in tests
- Match the `host` build backend to `pyproject.toml`'s `[build-system].requires`
- Always include `pip_check: true` in tests
- Use `--no-deps --no-build-isolation` in the pip install command
- Define `entry_points` that are defined in `[project.scripts]` in `pyproject.toml`

## With compiled extensions

```yaml
context:
  version: "1.0.0"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://files.pythonhosted.org/packages/source/e/example-package/example_package-${{ version }}.tar.gz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

build:
  number: 0
  script: python -m pip install . -vv --no-deps --no-build-isolation

requirements:
  build:
    - ${{ compiler('c') }}
    - ${{ stdlib('c') }}
    - if: build_platform != target_platform
      then:
        - cross-python_${{ target_platform }}
        - python
        - cython
        - numpy
  host:
    - python
    - pip
    - setuptools
    - cython
    - numpy
  run:
    - python
    - numpy

tests:
  - python:
      imports:
        - example_package
      pip_check: true

about:
  homepage: https://github.com/example-org/example-package
  summary: Short description of the package
  license: BSD-3-Clause
  license_file: LICENSE

extra:
  recipe-maintainers:
    - your-github-username
```

Key points:
- Cannot use `noarch: python`
- Add `${{ compiler('c') }}` and `${{ stdlib('c') }}` to build requirements
- Add cross-compilation support with conditional `build_platform != target_platform`

## abi3 extensions built with maturin

Extensions built against CPython's [stable ABI (`abi3`)](https://docs.python.org/3/c-api/stable.html) are compiled once against the minimum supported Python and reused for every later version, so the package can be shipped as Python version-independent instead of one build per Python version.

```yaml
context:
  version: "1.2.3"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://files.pythonhosted.org/packages/source/e/example-package/example_package-${{ version }}.tar.gz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

build:
  number: 0
  skip: is_abi3 and not is_python_min
  python:
    version_independent: ${{ is_abi3 }}
  script:
    env:
      CARGO_PROFILE_RELEASE_STRIP: symbols
      CARGO_PROFILE_RELEASE_LTO: fat
    content:
      # Remove this wrapper once https://github.com/conda-forge/rust-activation-feedstock/pull/79 is merged
      - if: unix
        then:
          - mkdir -p "${BUILD_PREFIX}/bin"
          - cp "${RECIPE_DIR}/cargo-auditable-wrapper.sh" "${BUILD_PREFIX}/bin/cargo-auditable-wrapper"
          - chmod +x "${BUILD_PREFIX}/bin/cargo-auditable-wrapper"
          - export CARGO="cargo-auditable-wrapper"
        else:
          - copy "%RECIPE_DIR%\cargo-auditable-wrapper.bat" "%BUILD_PREFIX%\Library\bin\cargo-auditable-wrapper.bat" || exit 1
          - set CARGO=cargo-auditable-wrapper.bat
      - cargo-bundle-licenses --format yaml --output THIRDPARTY.yml
      - python -m pip install . -vv --no-deps --no-build-isolation

requirements:
  build:
    - ${{ compiler('rust') }}
    - ${{ stdlib('c') }}
    - cargo-bundle-licenses
    - cargo-auditable
  host:
    - python
    - if: is_abi3
      then: python-abi3
    - pip
    - maturin
  run:
    - python

tests:
  - python:
      imports:
        - example_package
      pip_check: true
      python_version:
        - if: is_abi3
          then: ${{ python_min }}.*
        - "*"
  - if: is_abi3
    then:
      script:
        - if: win
          then: abi3audit %PREFIX%/Lib/site-packages/example_package/_native.pyd -s -v --assume-minimum-abi3 ${{ python_min }}
          else: abi3audit $SP_DIR/example_package/_native.abi3.so -s -v --assume-minimum-abi3 ${{ python_min }}
      requirements:
        run:
          - abi3audit

about:
  homepage: https://github.com/example-org/example-package
  summary: Short description of the package
  license: MIT
  license_file:
    - LICENSE
    - THIRDPARTY.yml

extra:
  recipe-maintainers:
    - your-github-username
```

Key points:
- `is_abi3` and `is_python_min` come from conda-forge's build matrix — no need to define them.
- `is_abi3` describes the Python *variant* being built against, not the package: regular CPython supports the stable ABI (`true`, build once against `python_min`), while free-threading CPython and PyPy don't (`false`, fall back to a normal per-Python build). The `if: is_abi3` selectors let one recipe cover both.
- `python-abi3` in host (under `if: is_abi3`) supplies the limited-API headers/link target and pins the abi3 toolchain — it does not by itself switch the extension to the stable ABI. The upstream Rust project has to opt in, by enabling PyO3's `abi3-py3XY` feature (or `abi3` plus a minimum version) in its `Cargo.toml`; otherwise maturin still builds a version-specific extension.
- `skip: is_abi3 and not is_python_min` keeps the abi3 build to a single Python version.
- Test the built extension with `abi3audit` to verify it only uses stable ABI symbols.
- If the package doesn't support abi3, drop every abi3-related piece and build it as a normal compiled extension.
- `cargo-auditable` cannot be invoked as `cargo auditable install` here because maturin calls `cargo` itself. Point `CARGO` at a wrapper placed next to `recipe.yaml` that forwards through it:

  ```sh title="cargo-auditable-wrapper.sh"
  #!/bin/sh
  exec cargo auditable $*
  ```

  ```bat title="cargo-auditable-wrapper.bat"
  @echo off
  cargo auditable %*
  ```

  These wrappers are temporary; once [rust-activation-feedstock#79](https://github.com/conda-forge/rust-activation-feedstock/pull/79) is merged, `cargo auditable` is wired up automatically and they can be removed.

Full example recipes in both formats live in [`python-abi3-feedstock`](https://github.com/conda-forge/python-abi3-feedstock): [`example-recipe.yaml`](https://github.com/conda-forge/python-abi3-feedstock/blob/main/recipe/example-recipe.yaml) (v1) and [`example-meta.yaml`](https://github.com/conda-forge/python-abi3-feedstock/blob/main/recipe/example-meta.yaml) (v0).
