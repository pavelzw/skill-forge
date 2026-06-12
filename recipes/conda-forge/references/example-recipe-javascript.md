# Example Recipe: JavaScript (Node.js / npm)

## Build Script (`build.sh`)

JavaScript packages require a separate build script. Place it next to `recipe.yaml`:

```bash
#!/usr/bin/env bash

set -o xtrace -o nounset -o pipefail -o errexit

# Create package archive and install globally from local source
tgz=$(npm pack --ignore-scripts)
npm install -ddd \
    --global \
    ${SRC_DIR}/${tgz}

# Create license report for dependencies
pnpm install
pnpm-licenses generate-disclaimer --prod --output-file=third-party-licenses.txt
```

## Recipe (`recipe.yaml`)

```yaml
context:
  name: example-package
  version: "1.0.0"

package:
  name: ${{ name }}
  version: ${{ version }}

source:
  url: https://registry.npmjs.org/@scope/example-package/-/example-package-${{ version }}.tgz
  sha256: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

build:
  number: 0
  skip:
    - win

requirements:
  build:
    - pnpm
    - pnpm-licenses
    - nodejs
  run:
    - nodejs

tests:
  - requirements:
      run:
        - nodejs
    script:
      - example-package --help

about:
  summary: Summary of the package
  license: MIT
  license_file:
    - LICENSE
    - third-party-licenses.txt
  homepage: https://github.com/example/example-package
  repository: https://github.com/example/example-package

extra:
  recipe-maintainers:
    - your-github-username
```

## Key Points

- **Always build from source**: Use `npm pack` + `npm install --global` to install from the local source tarball. Never repackage a pre-built binary.
- **Build script is required**: JavaScript recipes use a `build.sh` script (placed alongside `recipe.yaml`) instead of inline `build.script` in the recipe.
- **Source tarball**: Use the npm registry URL (`https://registry.npmjs.org/...`) for the source tarball, not GitHub. For scoped packages, the URL pattern is `@scope/package/-/package-version.tgz`.
- **Bundle third-party licenses**: Use `pnpm-licenses` to generate a `third-party-licenses.txt` and include it in `license_file`. This is the JavaScript equivalent of `cargo-bundle-licenses` for Rust.
- **Build requirements**: `pnpm`, `pnpm-licenses`, and `nodejs` go in `requirements.build`. Only `nodejs` is needed at runtime.
- **Skip Windows**: JavaScript CLI packages on conda-forge typically skip Windows (`skip: - win`).
- **Skip ppc64le**: npm packages often don't support ppc64le — add to `skip` if needed.
- **Tests**: For CLI packages, test with `--help`. Always include `nodejs` in test `requirements.run`.
