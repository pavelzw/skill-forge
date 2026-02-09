# Create a New Recipe

## Steps

### 1. Research the package

Before writing anything, gather information about the package:

- **Search prefix.dev (conda-forge)**: Use the GraphQL API to find existing packages and their dependencies:

  ```bash
  # Search for packages by name similarity
  curl -s -X POST 'https://prefix.dev/api/graphql' \
    -H 'Content-Type: application/json' \
    -d '{"query":"{ channel(name: \"conda-forge\") { packages(limit: 5, orderBy: {bySimilarity: {field: NAME, matches: \"PACKAGE_NAME\"}}) { page { name summary latestVersion { version } } } } }"}' | python3 -m json.tool

  # Get detailed info about a specific package (dependencies, license, urls)
  curl -s -X POST 'https://prefix.dev/api/graphql' \
    -H 'Content-Type: application/json' \
    -d '{"query":"{ package(channelName: \"conda-forge\", name: \"PACKAGE_NAME\") { name summary latestVersion { version platforms } urls { url kind } variants(limit: 1, platform: \"linux-64\") { page { version license rawIndex } } } }"}' | python3 -m json.tool
  ```

  The `rawIndex` field in variants contains the full dependency list under `depends`.
  The `urls` field with `kind: "FEEDSTOCK"` links to the conda-forge feedstock recipe.

- **Check Wolfi**: Search for the package in Wolfi's package repository (https://github.com/wolfi-dev/os) to see how they build it. Wolfi recipes use `melange` YAML format and are a great reference for C/C++ packages.
- **Check Homebrew**: Search for the formula at https://github.com/Homebrew/homebrew-core for build flags, dependencies, and patches.
- **Check Nixpkgs**: Search for the package derivation in Nixpkgs for build insights.
- **Check Yggdrasil**: https://github.com/JuliaPackaging/Yggdrasil/
- **Check upstream**: Find the official source (GitHub, GitLab, tarball URL) to understand build system (autotools, cmake, meson, etc.).

**Important**: Unlike Wolfi or Nix, rattler-build packages install to a `$PREFIX` directory, not to system paths. Always configure with `--prefix=$PREFIX` or equivalent.

### 2. Create the recipe directory

```bash
mkdir -p recipes/<package-name>
```

### 3. Write the recipe

Create `recipes/<package-name>/recipe.yaml`. See the recipe format reference in the main skill file.

### 4. Get the source hash

Download the source and compute sha256:

```bash
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
