# Creating New Conda-Forge Packages

## Overview

New packages are submitted via the [staged-recipes](https://github.com/conda-forge/staged-recipes) repository. After the PR is merged, conda-forge automatically creates a dedicated feedstock repository and CI infrastructure.

## Workflow

### 1. Generate the recipe

**Python packages** (preferred method):

```bash
pixi exec grayskull pypi --use-v1-format --strict-conda-forge <PACKAGE_NAME>
```

This produces a `recipe.yaml` that typically needs manual review and editing.

**Go/Rust**: Copy the appropriate template from [example-recipe-go.md](example-recipe-go.md) or [example-recipe-rust.md](example-recipe-rust.md) and adapt it.

**Other languages**: Create a recipe manually.

### 2. Fork and branch staged-recipes

```bash
gh repo fork conda-forge/staged-recipes --clone=true
```

```bash
git switch -c <PACKAGE_NAME>
```

### 3. Add the recipe

Create `recipes/<PACKAGE_NAME>/recipe.yaml` with the generated/adapted recipe.
Never touch files outside of the `recipe/` directory.

Important rules:
- Use tarballs with SHA256 checksums, not git clones
- Use SPDX license identifiers (e.g., `Apache-2.0`, `MIT`, `GPL-3.0-or-later`)
- Include a `license_file` reference
- If package uses static libraries, include those licenses as well (see go/rust example recipes)
- Tests are mandatory (at minimum, an import test for Python or a `--help` test for CLI tools)
- All dependencies must exist on conda-forge
- Remove any template/placeholder comments from the recipe

### 4. Compute the source hash

```bash
curl -Ls "<SOURCE_URL>" | sha256sum -
```

### 5. Test locally (optional but recommended)

From the staged-recipes root:
```bash
pixi exec rattler-build build -r recipes/<PACKAGE_NAME> -m .ci_support/<VARIANT_FILE>.yaml
```

Don't use `build-locally.py` as this introduces too much overhead.
Always invoke `rattler-build` directly.

### 6. Submit the PR

Create a draft PR (use the pull request template and check off everything you did).
Watch the CI until it is green.

In case you want to skip a platform, you can add something like `skip: win` in the `build` section of the recipe.
Once the CI is green, you are finished. Don't mark the PR as ready for review since the human that instructed you should do this manually.

## Multiple related packages

Submit interdependent packages in a single PR with separate directories under `recipes/`. The build system resolves dependency order within staged-recipes.

## Modify variant files

In case you need to adjust the c stdlib target version because of some missing APIs, you can create a `conda_build_config.yaml` file next to your recipe.
It can look like this:

```yaml
c_stdlib_version:  # [osx]
  - 12.0  # [osx]
```

Note the legacy `# [osx]` selectors are still from conda-build times but in the variant files, `rattler-build` can still read them for compatibility reasons.

## After feedstock creation

- Your human becomes a maintainer with commit rights to the feedstock
- Future changes go through PRs from forks (same as fixing builds)
- The `regro-cf-autotick-bot` auto-creates version update PRs

## Key decisions for the recipe

| Decision | Guidance |
|----------|----------|
| `noarch: python`? | Yes if pure Python, no compiled extensions, no platform-specific code |
| `noarch: generic`? | Yes if the package does not contain any compiled code (like shell-only recipes) |
| Build backend? | Match `pyproject.toml`'s `[build-system].requires` |
| Run dependencies? | Check `pyproject.toml`'s `[project].dependencies` |
| Test strategy? | Import test + `pip_check: true` at minimum; include unit tests if feasible |
| License? | Use SPDX identifier, include `license_file` |
