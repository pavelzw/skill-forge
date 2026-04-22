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

- Use source tarballs with SHA256 checksums (compute with `curl -Ls "<URL>" | sha256sum -`)
- Use SPDX license identifiers and include `license_file`
- Tests are mandatory (at minimum: import test for Python, `--help` for CLI)
- All dependencies must already exist on conda-forge
- Remove any template/placeholder comments

Test locally:

```bash
rattler-build build -r recipes/<PACKAGE_NAME> -m .ci_support/<VARIANT>.yaml
```

For `noarch: python` packages, provide a `python_min` variant override since staged-recipes doesn't define it:

```bash
rattler-build build -r recipes/<PACKAGE_NAME> -m .ci_support/<VARIANT>.yaml --variant python_min=3.10
```

Submit a draft PR, **always** use the PR template for the description. Watch CI until green. Don't mark as ready for review — let the human do that. To skip a platform, add `skip: win` in the `build` section.

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

1. Wait for CI: `pixi exec cf-job-logs wait-for-ci --json <PR_URL>`
   - Polls until all checks complete, then reports which passed/failed
   - Exits 0 if all pass, 1 if any fail — use this to gate further investigation
2. List failed jobs: `pixi exec cf-job-logs list-jobs --json <PR_URL>`
   - Returns a JSON array of jobs with `id`, `result`, `platform`, and `name` fields
   - The output only contains failed jobs by default; use `--all` to include successful jobs if needed
   - Pipe to `jq` for filtering if needed
3. Download a specific job's log: `pixi exec cf-job-logs download-log <PR_URL> <JOB_ID>`
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

### Python Version Pinning (noarch recipes)

For `noarch: python` recipes, use conda-forge's pinning conventions:

- host: `python ${{ python_min }}.*`
- run: `python >=${{ python_min }}`
- tests: `python_version: ${{ python_min }}.*`

If a newer Python minimum is required than conda-forge's default (3.10), override `python_min` in the `context` section of the recipe.

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

- [Python recipe template](references/example-recipe-python.md)
- [Go recipe template](references/example-recipe-go.md)
- [Rust recipe template](references/example-recipe-rust.md)
- [ARM builds and cross-compilation](references/arm-builds-and-cross-compilation.md)
- [Shell completions for CLI packages](references/shell-completions.md)
- [Recipe migration (v0 → v1)](references/rattler-build-migration.md)
