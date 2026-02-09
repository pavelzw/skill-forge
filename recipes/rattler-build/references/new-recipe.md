# Create a New Recipe

## Steps

### 1. Research the package

Before writing anything, gather information about the package:

- **Search conda-forge**: Use `pixi search` to find existing packages:

  ```bash
  # Search for packages by name
  pixi search 'PACKAGE_NAME*' -c conda-forge

  # Get detailed info about a specific package (dependencies, version, etc.)
  pixi search PACKAGE_NAME -c conda-forge --verbose
  ```

- **Check Wolfi**: Search for the package in Wolfi's package repository (https://github.com/wolfi-dev/os) to see how they build it. Wolfi recipes use `melange` YAML format and are a great reference for C/C++ packages.
- **Check Homebrew**: Search for the formula at https://github.com/Homebrew/homebrew-core for build flags, dependencies, and patches.
- **Check Nixpkgs**: Search for the package derivation in Nixpkgs for build insights.
- **Check Yggdrasil**: https://github.com/JuliaPackaging/Yggdrasil/
- **Check upstream**: Find the official source (GitHub, GitLab, tarball URL) to understand build system (autotools, cmake, meson, etc.).

**Important**: Unlike Wolfi or Nix, rattler-build packages install to a prefix directory, not to system paths. Always configure with `--prefix=$PREFIX` or equivalent.

> **`$PREFIX` vs `${{ PREFIX }}`**: Use the shell variable `$PREFIX` inside build scripts
> (e.g., `./configure --prefix=$PREFIX`). Use the Jinja expression `${{ PREFIX }}` inside
> YAML fields (e.g., `build.script` list entries in recipe.yaml).

### 2. Create the recipe directory

```bash
mkdir -p recipes/<package-name>
```

### 3. Write the recipe

Create `recipes/<package-name>/recipe.yaml`. See the recipe format reference in the main skill file.

### 4. Get the source hash

Download the source and compute sha256:

```bash
# On most Linux systems:
curl -sL <source-url> | sha256sum

# On macOS or systems with shasum:
curl -sL <source-url> | shasum -a 256
```

### 5. Build and test

```bash
rattler-build build --recipe recipes/<package-name>/recipe.yaml --keep-build
```

### 6. Iterate on failures

If the build fails, use the debug-build workflow to investigate and fix issues.

## Guidelines

- **Build dependencies** go in `requirements.build`: compilers (`${{ compiler('c') }}`), build tools (cmake, ninja, make, pkg-config)
- **Host dependencies** go in `requirements.host`: libraries to link against (zlib, openssl, etc.)
- **Run dependencies** go in `requirements.run`: runtime libraries, interpreters
- Always use `${{ PREFIX }}` for install prefix, never hardcode paths
- Use `${{ CPU_COUNT }}` for parallel make jobs
- Include both `package_contents` and `script` tests
