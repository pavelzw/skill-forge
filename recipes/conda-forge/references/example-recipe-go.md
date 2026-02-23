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
    - if: unix
      then:
        - mkdir -p $PREFIX/share/zsh/site-functions $PREFIX/share/bash-completion/completions $PREFIX/share/fish/vendor_completions.d
        - $PREFIX/bin/example-package completion --shell zsh > $PREFIX/share/zsh/site-functions/_example-package
        - $PREFIX/bin/example-package completion --shell bash > $PREFIX/share/bash-completion/completions/example-package
        - $PREFIX/bin/example-package completion --shell fish > $PREFIX/share/fish/vendor_completions.d/example-package.fish

requirements:
  build:
    - ${{ compiler("go-nocgo") }}
    - go-licenses

tests:
  - script: example-package -help
  - package_contents:
      bin:
        - example-package
      files:
        - if: unix
          then:
            - share/bash-completion/completions/example-package
            - share/fish/vendor_completions.d/example-package.fish
            - share/zsh/site-functions/_example-package
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
- Generate shell completions for bash, zsh, and fish on Unix platforms. This assumes the binary supports a `completion --shell <shell>` subcommand. The completion files are installed to the standard locations under `$PREFIX/share/`. If your tool does not support generating completions, remove those build and test lines.
- Verify that both the binary and the shell completion files are present using `package_contents` tests, with `strict: true` to ensure no unexpected files are installed.

In case a license is missing, use `--skip ...` to skip the package but ensure it's still part of the package by manually downloading it from `https://raw.githubusercontent.com/.../refs/heads/master/LICENSE`:

```yaml
source:
  - url: https://github.com/example-package/example-package/archive/refs/tags/v${{ version }}.tar.gz
    sha256: ...
    target_directory: src
  - url: https://raw.githubusercontent.com/.../refs/heads/master/LICENSE
    sha256: ...
    file_name: LICENSE-other-package

# ...

about:
  # ...
  license: MIT
  license_file:
    - src/LICENSE
    - library_licenses/
    - LICENSE-other-package
```

You might need to set the following in `conda-forge.yml` to ensure that the `$PREFIX/bin/example-package completion --shell ...` steps work:

```yml
provider:
  osx_arm64: azure
```

Generating the completions only works when running the native binary which doesn't work with `osx-64 -> osx-arm64` cross-compilation.
