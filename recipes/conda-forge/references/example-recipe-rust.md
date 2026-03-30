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
