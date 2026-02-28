# Rattler-Build Migration (v0 to v1 Recipe Format)

## Overview

conda-forge is migrating recipes from v0 format (`meta.yaml`) to v1 format (`recipe.yaml`) for use with rattler-build. The migration has two approaches: automated (preferred) and manual (fallback).

## Approach 1: Automated with feedrattler (preferred)

[feedrattler](https://github.com/hadim/feedrattler) automates the conversion and creates a draft PR.

### Run

```bash
pixi exec feedrattler <FEEDSTOCK_NAME>
```

For example:
```bash
pixi exec feedrattler numpy-feedstock
```

This will:
1. Fork the feedstock (if needed)
2. Clone it locally
3. Convert `meta.yaml` to `recipe.yaml` using Conda Recipe Manager
4. Rerender the feedstock
5. Push to your fork
6. Create a draft PR on the upstream feedstock

### Key options

| Flag | Default | Description |
|------|---------|-------------|
| `--rerender / --no-rerender` | enabled | Rerender after conversion |
| `--draft-pr / --no-draft-pr` | enabled | Create draft PR |
| `--branch-name TEXT` | `convert_feedstock_to_v1_recipe_format` | Custom branch name |
| `--local-clone-dir TEXT` | - | Use existing local clone |
| `--log-level TEXT` | INFO | DEBUG, INFO, WARNING, ERROR |

### Authentication

feedrattler uses credentials in this order:
1. `GITHUB_TOKEN` environment variable
2. GitHub CLI (`gh`) configured credentials
3. `--github-token` flag

### After feedrattler runs

1. Check the draft PR for CI status
2. If CI passes, mark the PR as ready for review
3. If CI fails, diagnose and fix the issues (see below)

## Approach 2: Manual migration (fallback)

Use this when feedrattler fails or produces an incorrect conversion.

### Steps

1. Clone the feedstock (from your fork)
2. Read the existing `meta.yaml`
3. Create a new `recipe.yaml` following v1 format
4. Add `conda_build_tool: rattler-build` and `conda_install_tool: pixi` to `conda-forge.yaml`
5. Delete the old `meta.yaml`
6. Commit those changes
7. Rerender: `pixi exec conda-smithy rerender --commit=all`
8. Lint the recipe: `pixi exec conda-smithy lint --feedstock-dir . recipe/`
9. Test locally: `pixi exec rattler-build build --recipe recipe --variant-config .ci_support/<VARIANT>.yaml` (depends on your system)
10. Submit a draft PR from your fork and check the CI.
11. If CI passes, mark the PR as ready for review.
12. If CI fails, diagnose and fix the issues (see below).

### Key v0 to v1 differences

| Feature | v0 (`meta.yaml`) | v1 (`recipe.yaml`) |
|---------|-------------------|---------------------|
| Jinja syntax | `{{ version }}` | `${{ version }}` |
| Variables | `{% set version = "1.0" %}` | `context:` block |
| Selectors | `# [win]` comments | `if:` / `then:` / `else:` |
| Compiler | `{{ compiler('c') }}` | `${{ compiler('c') }}` |
| Pin compatible | `{{ pin_compatible('pkg') }}` | `${{ pin_compatible('pkg') }}` |
| Test python imports | `test.imports` | `tests[].python.imports` |
| Test commands | `test.commands` | `tests[].script` |
| Package contents tests | not available | `tests[].package_contents` |
| Build script | `build.script` | `build.script` (list of strings) |
| `run_exports` location | `build.run_exports` | `requirements.run_exports` |
| `run_constrained` | `requirements.run_constrained` | `requirements.run_constraints` |
| `ignore_run_exports` | `build.ignore_run_exports` | `requirements.ignore_run_exports.by_name` |
| `skip` syntax | `build: skip: true  # [win]` | `build: skip: [win]` |
| `about` home field | `about.home` | `about.homepage` |
| Build string hash | `h{{ PKG_HASH }}` | `h${{ hash }}` |

### Common conversion pitfalls

- Forgetting to convert Jinja `{{ }}` to `${{ }}`
- Selector syntax: `# [win]` becomes
```yaml
- if: win
  then:
    - ...
  # optional
  else:
    - ...
```
- Test section restructuring: v1 uses a list of test objects
- `pin_subpackage` and `pin_compatible` syntax changes
- `skip` syntax changed: `build: skip: true  # [win]` becomes
```yaml
build:
  skip:
    - win
```
See https://rattler.build/latest/reference/recipe_file/#skipping-builds for details.
- `run_exports` moved from `build:` to `requirements:`, and `max_pin` renamed to `upper_bound`:
```yaml
# v0
build:
  run_exports:
    - {{ pin_subpackage('pkg', max_pin='x') }}
# v1
requirements:
  run_exports:
    - ${{ pin_subpackage('pkg', upper_bound='x') }}
```
- `run_constrained` renamed to `run_constraints`
- `ignore_run_exports` moved from `build:` to `requirements:` with restructured keys:
```yaml
# v0
build:
  ignore_run_exports:
    - zlib
  ignore_run_exports_from:
    - zlib
# v1
requirements:
  ignore_run_exports:
    by_name:
      - zlib
    from_package:
      - zlib
```
- Remove `pip check` from test commands / scripts — it is automatically handled inside `tests[].python.imports:`
- `about.home` renamed to `about.homepage` (required by the conda-forge linter)
- `PKG_HASH` replaced by `hash`; `PKG_BUILDNUM` has no direct equivalent (define a context variable instead):
```yaml
# v0
build:
  string: {{ string_prefix }}_h{{ PKG_HASH }}
# v1
build:
  string: ${{ string_prefix }}_h${{ hash }}
```
- Keep using implicit `build.sh` / `build.bat` scripts rather than embedding commands in the recipe; note that on Windows the implicit script name changed from `bld.bat` (v0) to `build.bat` (v1) — call it explicitly to avoid renaming it
- noarch python recipes: use `python_min` from the conda-forge pinning for host/run/test constraints
  - host: `- python ${{ python_min }}.*`
  - run: `- python >=${{ python_min }}`
  - test `python_version`: `${{ python_min }}.*`
