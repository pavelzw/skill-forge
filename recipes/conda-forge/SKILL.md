---
name: conda-forge
description: Performs conda-forge operations. Fixes failing builds by analyzing CI logs, creates new packages via staged-recipes, adds native ARM CI support or explicit cross-compilation to feedstocks, and migrates recipes from v0 to v1 format. Use when working with conda-forge feedstocks, staged-recipes, build failures, recipe migrations, or when the user mentions conda-forge.
license: BSD-3-Clause
compatibility: >-
  Requires gh (GitHub CLI), git, curl, jq, tar, and pixi with
  rattler-build, conda-smithy, and cf-job-logs. Requires network access.
---

# Conda-Forge Skill

First, determine the workflow by checking the current working directory:
- **staged-recipes** repo → you are creating a new feedstock
- **\<name\>-feedstock** repo → you are updating an existing feedstock

## Staged Recipes: Creating New Feedstocks

New packages are submitted via [staged-recipes](https://github.com/conda-forge/staged-recipes). After merge, conda-forge auto-creates a dedicated feedstock.

Fork and branch:

```bash
gh repo fork conda-forge/staged-recipes --clone=true
cd staged-recipes
git switch -c <PACKAGE_NAME>
```

Generate a recipe for Python packages with `pixi exec grayskull pypi --use-v1-format --strict-conda-forge <PACKAGE_NAME>`. Since grayskull creates the `my-package/recipe.yaml` in the CWD, run this command in the `recipes/` directory of your staged-recipes fork.

For Go or Rust, adapt the templates from [example-recipe-go.md](references/example-recipe-go.md) or [example-recipe-rust.md](references/example-recipe-rust.md). See also [example-recipe-python.md](references/example-recipe-python.md).

Place the recipe in `recipes/<PACKAGE_NAME>/recipe.yaml`.

Recipe rules:

- Use [official source archives](#choosing-source-archives) with SHA256 checksums.
- Use SPDX license identifiers and include `license_file`
- Tests are mandatory (at minimum: import test for Python, `--help` for CLI)
- All dependencies must already exist on conda-forge
- Remove any template/placeholder comments

Test locally:

```bash
rattler-build build -r recipes/<PACKAGE_NAME> -m .ci_support/<VARIANT>.yaml
```

For `noarch: python` or `abi3` packages, use the effective conda-forge variant configuration. A platform config alone may not supply the Python pins and selectors used in CI. If `python_min` is missing for a local noarch build, set `PYTHON_MIN` to the applicable minimum and pass it explicitly:

```bash
rattler-build build -r recipes/<PACKAGE_NAME> -m .ci_support/<VARIANT>.yaml --variant "python_min=$PYTHON_MIN"
```

Submit a draft PR, **always** use the PR template for the description. Base the static-library checklist on [the built package contents](#static-libraries-and-bundled-code). In your PR description, never `@`-mention anybody!
Only submit a non-draft PR on staged recipes as undrafting re-triggers CI. On feedstocks, undrafting pings people which you should avoid.
Watch CI until green. Don't mark as ready for review — let the human do that. To skip a platform, add `skip: win` in the `build` section.

For submitting multiple related packages, place each in a separate directory under `recipes/`. The build system resolves dependency order within staged-recipes.

## Existing Feedstock: Updating

Work on `recipe/recipe.yaml`. If you only find a `meta.yaml`, you must convert it to `recipe.yaml` first — see [rattler-build-migration.md](references/rattler-build-migration.md).

### Critical: Always Use a Personal Fork

conda-forge requires all PRs to come from forks, NOT from branches in the main feedstock repository.

If you push directly to `conda-forge/<feedstock>`, it will try to upload to conda-forge directly which is not allowed!

You MUST:

1. Fork the feedstock to your personal GitHub account (or use an existing fork)
2. Push your fix branch to YOUR fork
3. Create a PR from <YOUR_USERNAME>:<branch> to conda-forge/<feedstock>:main

Use `gh repo fork conda-forge/<feedstock> --clone` to fork the feedstock.
This will clone the forked repository with the origin remote pointing to your fork and the upstream remote pointing to the original feedstock.

**Important**: If the fork already existed, its `main` branch may be out of date with upstream. Always sync it before creating your fix branch. First back up the fork's current `main` in case it has useful commits, then reset to upstream:

```bash
git fetch upstream
git checkout main
git branch main-fork-backup
git push origin main-fork-backup
git reset --hard upstream/main
git push origin main --force
```

### Fixing Failing Builds

Start by reproducing the issue locally — run a local build (see Test Locally below) and iterate from there. If `rattler-build` fails, it keeps the work directory at `output/bld/rattler-build_.../work` — you can debug with `cd <work> && source build_env.sh`.

If the user explicitly references CI failures or pastes a link, use `cf-job-logs` to investigate:

1. Wait for CI: `pixi exec cf-job-logs --gh wait-for-ci --json <PR_URL>`
   - Polls until all checks complete, then reports which passed/failed
   - Exits 0 if all pass, 1 if any fail — use this to gate further investigation
   - `--fail-fast` (default: true) returns as soon as any check fails instead of waiting for the rest — use it when you want to iterate on a single specific error. Pass `--no-fail-fast` to wait for every check to finish when you want to confirm all results up front and then collect all errors at once
2. List failed jobs: `pixi exec cf-job-logs --gh list-jobs --json <PR_URL>`
   - Returns a JSON array of jobs with `id`, `result`, `platform`, and `name` fields
   - The output only contains failed jobs by default; use `--all` to include successful jobs if needed
   - Pipe to `jq` for filtering if needed
3. Download a specific job's log: `pixi exec cf-job-logs --gh download-log <PR_URL> <JOB_ID>`
   - Use the `id` from the `list-jobs` output as `<JOB_ID>`
   - Logs are sanitized by default (timestamps and known boilerplate removed; `--no-sanitize` is available but rarely needed)
   - Redirect to a file with `> log.txt` for large logs
4. Read the error log — understand the root cause before making any changes

**Recommended workflow**: If CI is still running, start with `wait-for-ci` to block until completion. Then use `list-jobs` to get the job IDs for any failures, and `download-log` to fetch the actual logs.

Run `pixi exec cf-job-logs --help` or `pixi exec cf-job-logs <command> --help` for the full list of options.

Apply the minimal fix needed. Only modify files in the `recipe/` directory.

### Test Locally

Always test with a variant config that matches your local platform:

```bash
rattler-build build --recipe recipe -m .ci_support/<VARIANT>.yaml
```

### Finalize Changes

After all recipe changes, always run:

```bash
pixi exec conda-smithy rerender --no-check-uptodate --commit=auto
pixi exec conda-smithy lint --conda-forge .
```

Commit your recipe changes first, then rerender (which auto-commits if there are changes). Only modify files in the `recipe/` directory — everything else is autogenerated by rerender.

Push to your fork and create a PR to `conda-forge/<FEEDSTOCK>:main`.

## General

If a tool isn't available locally, use `pixi exec <tool>` to run tools (rattler-build, conda-smithy, grayskull, feedrattler) without installing them. Use `mktemp -d` for temporary working directories.

### Choosing Source Archives

- For Python packages, inspect the published PyPI source distribution (sdist) first. Compare it with the upstream release archive if needed: make sure the chosen archive contains the required sources, headers, and build inputs, and check whether it adds bundled dependencies or runtime data that change package behavior.
- If an archive omits a Git submodule, use the submodule's exact commit from the parent release, not its latest tag or branch. When needed, fetch a separate archive with `target_directory` set to the submodule path. If the sdist already contains the submodule code and only lacks its license, fetch just that license from the same commit.
- Pin a SHA256 for every download, including standalone license files. Recheck the submodule revision and checksums when updating the package.

### Python Version Pinning (noarch recipes)

For `noarch: python` recipes, use conda-forge's pinning conventions:

- host: `python ${{ python_min }}.*`
- run: `python >=${{ python_min }}`
- tests: `python_version: [${{ python_min }}.*, "*"]` (minimum and latest Python)

If the package requires a newer minimum than the current conda-forge pin, override `python_min` in the `context` section of the recipe.

### Python Stable ABI (`abi3`)

An upstream `abi3` wheel doesn't make a conda package version-independent. First verify that the source build actually targets CPython's stable ABI. Then follow the [conda-forge `abi3` recipe pattern](https://conda-forge.org/docs/maintainer/example_recipes/abi3/): use conditional `python-abi3` in `host`, set `build.python.version_independent` from `is_abi3`, and skip `is_abi3 and not is_python_min`. Keep non-`abi3` variants interpreter-specific where upstream supports them.

This still produces a separate binary for each OS and architecture. Test the same built package on the minimum and a newer supported CPython, and use `abi3audit` to check the extension's symbols. See the [Python example](references/example-recipe-python.md#abi3-extensions-built-with-maturin) for how to express the tests in a recipe.

### Patching Sources

If the upstream source needs changes, use git-style patches (`git format-patch`) with a commit message explaining why the patch is needed. Put them in `recipe/patches/` and list them under `source.patches`.

Keep patches focused on the packaging requirement. Leave unrelated refactors, formatting, and cleanup alone, even if an import or helper becomes unused. Verify that the patch applies to the exact source archive used by the recipe.

### Validating Platform-Specific Builds

Rendering a recipe checks the variant configuration, not compilation or linking. Test affected paths on the target platform with the CI toolchain and host dependencies when feasible. If a native build isn't available, use a supported cross-build or run the affected preprocessor, parser, or generator against the actual target headers. Say which parts, such as linking or runtime tests, remain unverified.

### Static Libraries and Bundled Code

Inspect the built package's file list before answering the static-library checklist. Code compiled into a shared extension or executable is not itself a shipped static archive. Windows import libraries are also distinct from static libraries. If static archives must be shipped, follow [CFEP-18](https://github.com/conda-forge/cfep/blob/main/cfep-18.md), which generally calls for a separate `-static` output.

Check vendoring separately: prefer packaged runtime dependencies where practical, and include the required licenses for any bundled or statically linked code.

### Common Errors

- **`no candidates were found`**: Wrong dependency name. Use `pixi search <name>` (supports globs like `pixi search 'lib*'`).
- **Import errors in test phase**: Missing runtime dependency → add to `requirements.run`.
- **Compilation failure**: Missing library → add to `requirements.host`.
- **Command not found (exit 127)**: Missing build tool → add to `requirements.build`.

### Key Decisions

- `noarch: python`: Use for pure Python packages with no compiled extensions or platform-specific code.
- `noarch: generic`: Use for packages without any compiled code (e.g., shell-only recipes).
- Build backend: Match `pyproject.toml`'s `[build-system].requires`.
- For ARM enablement and explicit cross-compilation details, see [arm-builds-and-cross-compilation.md](references/arm-builds-and-cross-compilation.md).

### References

- [Python recipe template](references/example-recipe-python.md) (pure Python, compiled extensions, maturin + abi3)
- [Go recipe template](references/example-recipe-go.md)
- [Rust recipe template](references/example-recipe-rust.md)
- [ARM builds and cross-compilation](references/arm-builds-and-cross-compilation.md)
- [Shell completions for CLI packages](references/shell-completions.md)
- [Recipe migration (v0 → v1)](references/rattler-build-migration.md)
