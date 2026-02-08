# Example Recipe: Python

## Pure Python (noarch)

```yaml
context:
  version: "1.0.0"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://pypi.org/packages/source/e/example-package/example_package-${{ version }}.tar.gz
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
    - python >=3.9
    - pip
    - hatchling
  run:
    - python >=3.9
    - requests
    - click

tests:
  - python:
      imports:
        - example_package
      pip_check: true
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
  url: https://pypi.org/packages/source/e/example-package/example_package-${{ version }}.tar.gz
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
