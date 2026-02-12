---
name: conda-forge
description: Performs conda-forge operations. Fixes failing builds by analyzing CI logs, creates new packages via staged-recipes, adds cross-compilation and ARM support to feedstocks, and migrates recipes from v0 to v1 format. Use when working with conda-forge feedstocks, staged-recipes, build failures, recipe migrations, or when the user mentions conda-forge.
license: BSD-3-Clause
compatibility: >-
  Requires gh (GitHub CLI), git, curl, jq, tar, and pixi with
  rattler-build and conda-smithy. Requires network access.
---

# Conda-Forge Operations

Determine which workflow applies based on the task:

| Task | Workflow |
|------|----------|
| Fix a failing CI build | [Fix Build](#fix-failing-build) (below) |
| Create a new conda-forge package | [references/creating-packages.md](references/creating-packages.md) |
| Add ARM / cross-compilation support | [references/cross-compilation.md](references/cross-compilation.md) |
| Migrate recipe from v0 to v1 | [references/rattler-build-migration.md](references/rattler-build-migration.md) |
| Need a Python recipe template | [references/example-recipe-python.md](references/example-recipe-python.md) |
| Need a Go recipe template | [references/example-recipe-go.md](references/example-recipe-go.md) |
| Need a Rust recipe template | [references/example-recipe-rust.md](references/example-recipe-rust.md) |

For allowed commands, see [references/allowed-commands.md](references/allowed-commands.md).
For common error patterns, see [references/common-issues.md](references/common-issues.md).

---

## Critical: Always Use a Personal Fork

conda-forge requires all PRs to come from forks, NOT from branches in the main feedstock repository.

If you push directly to `conda-forge/<feedstock>`, it will try to upload to `conda-forge` directly which is not allowed!

You MUST:
1. Fork the feedstock to your personal GitHub account (or use an existing fork)
2. Push your fix branch to YOUR fork
3. Create a PR from `<YOUR_USERNAME>:<branch>` to `conda-forge/<feedstock>:main`

Use `gh repo fork conda-forge/<feedstock> --clone` to fork the feedstock.
This will clone the forked repository with the `origin` remote pointing to your fork and the `upstream` remote pointing to the original feedstock.

---

# Fix Failing Build

Fix the failing conda-forge build at the provided PR URL.

Analyze the failing pull request, identify the build error, fix it, and submit a PR with the fix. If you can push to the fork directly, do so.

Execute each phase in order. Do not skip phases.

In case you are missing tools, don't install them but use `pixi exec <tool>` to run them without installation.

Use `mktemp -d` to create a temporary directory for storing intermediate files (and clone repositories there unless otherwise specified).

## Phase 1: Gather Information

**Objective**: Understand what failed and why.

### Step 1.1: Get PR metadata

```bash
gh pr view <PR_NUMBER> --repo <OWNER/REPO> --json headRefName,headRepository,statusCheckRollup,url,title
```

### Step 1.2: Find failing CI check
- Look for `statusCheckRollup` entries with `"conclusion": "FAILURE"` or `"state": "FAILURE"`
- Extract the `detailsUrl` (Azure Pipelines URL)
- Extract `buildId` from the URL (e.g., `buildId=1426205`)

### Step 1.3: Fetch build timeline

```bash
curl -s "https://dev.azure.com/conda-forge/feedstock-builds/_apis/build/builds/<BUILD_ID>/timeline?api-version=6.0" > $TMPFILE
```

### Step 1.4: Find failed step

```bash
jq '.records[] | select(.result == "failed") | {name, id, log: .log.url}' $TMPFILE
```

### Step 1.5: Fetch error log

```bash
curl -s "<LOG_URL>" > $TMPFILE
```

### Step 1.6: Read error log

Read the last 200 lines of `$TMPFILE`.

**Checkpoint**: You must understand the root cause before proceeding.

---

## Phase 2: Clone and Verify

**Objective**: Get the code locally and set up fork for pushing.

### Step 2.1: Clone YOUR FORK (not the main repo)

```bash
cd $TMPFILE && gh repo fork conda-forge/<FEEDSTOCK> --clone
```

### Step 2.2: Checkout PR branch

```bash
gh pr checkout <PR_NUMBER>
```

### Step 2.3: Verify remotes

```bash
git remote -v
```

Confirm `origin` points to YOUR fork, `upstream` to `conda-forge/<FEEDSTOCK>`.

### Step 2.4: Read recipe and verify schema version

- Read `recipe/recipe.yaml` or `recipe.yaml`

**STOP**: If there is only `meta.yaml` but not `recipe.yaml`, inform the user and halt. This workflow only supports v1 recipe format.

---

## Phase 3: Diagnose and Fix

**Objective**: Apply the minimal fix to resolve the build error.

### Step 3.1: Analyze the error

Based on Phase 1 findings, identify the category. See [references/common-issues.md](references/common-issues.md) for common patterns.
If the error is trivial, fix it directly in the recipe and push the fix.

### Step 3.2: Investigate if needed

If the error does not seem trivial, try to first reproduce the error locally.
If you are able to reproduce the error locally, iterate locally until you have found a working solution.
Then, push the fix and watch the CI status.

If `rattler-build` fails, it keeps the work directory for debugging purposes. It can be found in `output/bld/rattler-build_.../work` (see end of the `rattler-build build` output).
You may investigate the work directory or try to reproduce the error without retrying `rattler-build build` if the command takes long.
You can do so by running `cd <work> && source build_env.sh && ...`.

### Step 3.3: Apply the fix

Use the Edit tool to modify `recipe.yaml`. Apply ONLY the minimal change needed.

Do NOT make unrelated changes, add extra dependencies, refactor, or add comments.

ONLY modify files in the recipe directory. All other directories are autogenerated. Use `conda-smithy rerender --commit=auto` (Phase 5) for those.

---

## Phase 4: Test Locally

**Objective**: Verify the fix works before submitting.

### Step 4.1: List variant configs

```bash
ls .ci_support/
```

### Step 4.2: Run build

Select the machine you are running on.

```bash
pixi exec rattler-build build --recipe recipe --variant-config .ci_support/<VARIANT>.yaml
```

### Step 4.3: If build fails, iterate

Analyze the error, apply additional fixes, re-run.

**Checkpoint**: Build must succeed before proceeding.

---

## Phase 5: Lint and Rerender

### Step 5.1: Lint

```bash
pixi exec conda-smithy lint --conda-forge .
```

Fix any linting errors and re-lint.

For `noarch: python` recipes, do not pin Python by writing only a bare version number. Use conda-forge's expected pins:

- `host` section: `python ${{ python_min }}.*`
- `run` section: `python >=${{ python_min }}`
- Tests (`tests[].python.python_version` or `tests[].requirements.run`): `python_version: ${{ python_min }}.*` or `python ${{ python_min }}.*`
- If a newer Python minimum is required than conda-forge's default (3.10), override `python_min` at the top of the recipe in the `context` section

Building directly might not work in the staged-recipes repository because there, you don't specify `python_min`. You can inject additional context into rattler-build's build by using `--context python_min=3.10`.

### Step 5.2: Commit manual changes

Commit any manual changes you performed in `recipe/`.

### Step 5.3: Rerender after committing manual changes

```bash
pixi exec conda-smithy rerender --no-check-uptodate --commit=auto
```

This will autogenerate files and also commit them (if there are changes).

---

## Phase 6: Submit changes

**Objective**: Create a PR with the fix FROM YOUR FORK.

### Step 6.1: Push

In case you have write access to the fork of the failing CI, you can push directly to the fork.

In case you don't have write access, push to your fork:

```bash
git push -u origin <BRANCH_NAME>
```

`origin` must point to your fork, NOT to `conda-forge/<FEEDSTOCK>`.

Then, create a PR from your fork to the upstream repository.

### Step 6.2: Verify CI

Check the CI status on the PR. Ensure all checks pass.
If they don't, investigate the failures and fix them.

---

## Phase 7: Report

Provide a summary:

1. **Error found**: Root cause
2. **Fix applied**: What changed
3. **Test results**: Local build outcome
4. **Linting**: Pass/fail
5. **Rerender**: Applied or not
6. **PR URL**: Link to the submitted PR
7. **PR Changes URL**: <PR URL>/changes for easy diff views
