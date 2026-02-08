# Example Recipe: Go

```yaml
context:
  version: "0.1.0"

package:
  name: example-package
  version: ${{ version }}

source:
  url: https://github.com/example-package/example-package/archive/refs/tags/v${{ version }}.tar.gz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
  target_directory: src

build:
  number: 0
  script:
    - cd src
    - go-licenses save . --save_path ../library_licenses
    - if: unix
      then: go build -v -o $PREFIX/bin/example-package -ldflags="-s -w"
      else: go build -v -o %LIBRARY_BIN%\example-package.exe -ldflags="-s"

requirements:
  build:
    - ${{ compiler("go-nocgo") }}
    - go-licenses

tests:
  - script: example-package -help
  - package_contents:
      bin:
        - example-package
      strict: true

about:
  homepage: https://github.com/example-package/example-package
  summary: Summary of the package
  description: |
    Description of the package
  license: MIT
  license_file:
    - src/LICENSE
    - library_licenses/
  documentation: https://pkg.go.dev/github.com/example-package/example-package
  repository: https://github.com/example-package/example-package

extra:
  recipe-maintainers:
    - your-github-username
```

Key points:
- Use `${{ compiler("go-nocgo") }}` for static binaries (no cgo)
- Use `${{ compiler("go-cgo") }}` if the package uses cgo
- Use `go-licenses` to collect dependency licenses
- Place source in `target_directory: src` to keep license output separate
