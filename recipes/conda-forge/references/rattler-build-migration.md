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

1. feedrattler deletes `meta.yaml` and creates `recipe.yaml` in a single commit, which breaks `git blame` — split it as described in [Preserving git history](#preserving-git-history) and force-push to the PR branch
2. Check the draft PR for CI status
3. If CI passes, mark the PR as ready for review
4. If CI fails, diagnose and fix the issues (see below)

## Approach 2: Manual migration (fallback)

Use this when feedrattler fails or produces an incorrect conversion.

### Steps

1. Clone the feedstock (from your fork)
2. Rename the recipe file and commit it on its own, without changing its contents: `git mv recipe/meta.yaml recipe/recipe.yaml && git commit -m "rename recipe file"` (see [Preserving git history](#preserving-git-history))
3. Rewrite `recipe/recipe.yaml` in place following v1 format — for complicated multi-output recipes, consider one commit per output
4. Add `conda_build_tool: rattler-build` and `conda_install_tool: pixi` to `conda-forge.yaml`
5. Commit those changes
6. Rerender: `pixi exec conda-smithy rerender --commit=all`
7. Lint the recipe: `pixi exec conda-smithy lint --feedstock-dir . recipe/`
8. Test locally: `pixi exec rattler-build build --recipe recipe --variant-config .ci_support/<VARIANT>.yaml` (depends on your system)
9. Submit a draft PR from your fork and check the CI.
10. If CI passes, mark the PR as ready for review.
11. If CI fails, diagnose and fix the issues (see below).

### Preserving git history

Deleting `meta.yaml` and creating `recipe.yaml` in the same commit makes git register a full deletion and recreation, so `git blame` on `recipe.yaml` ends at one mega commit and years of accumulated history become hard to reach. From H. Vetinari on the [conda-forge Zulip](https://conda-forge.zulipchat.com/#narrow/channel/457337-general/topic/git-blame-friendly.20v1.20translation.20of.20a.20complicated.20feedstock/near/636474480):

> you get a lot of mileage already out of a trivial rename commit like https://github.com/conda-forge/clangdev-feedstock/commit/94b82daee49fef4e1b6c9392008c4b5037c2b143 before beginning with the conversion in earnest (because git then will keep the intra-file diff and blame history despite the file rename, rather than registering it as a full deletion & recreation).

> regular blame traversal in the GH UI works as expected [...] In other words, we can just transparently step through to how the recipe used to look like before the conversion, with a single click, and keeping all the years of accumulated history easily accessible.

So: **always** make the first commit a pure rename (`git mv recipe/meta.yaml recipe/recipe.yaml`, contents unchanged), then convert in subsequent commits.

For complicated, central multi-output feedstocks, go further and convert one output per commit, as in [clangdev-feedstock#470](https://github.com/conda-forge/clangdev-feedstock/pull/470) ("rename recipe file", "v1 transformation: context & extra", "v1 transformation: source", then "v1 transformation: <name> output" for each output):

> each output gets its own conversion commit, which makes reviewing the conversion after the fact much less of a hassle (otherwise you have to scroll multiple pages for the right context; one output seemed like the smallest reasonable unit to counteract this) [...] it's still *much* easier to see whether all requirements/files/tests etc. were translated correctly (especially if something went wrong and you're trying to determine where the bug was introduced).

> conda-forge isn't the linux kernel, not every commit is expected to be runnable, and bisections are extremely rare [...] This is likely overkill for most other conversions

Intermediate commits therefore don't need to build. Mention in the PR description that the commit history is intentional so maintainers merge with a merge commit rather than squashing (squashing would collapse it back into one delete-and-recreate commit).

#### Splitting a feedrattler commit

feedrattler produces one `Convert to v1 feedstock...` commit (optionally followed by a rerender commit). To insert a rename commit before it without changing the final tree:

```bash
B=convert_feedstock_to_v1_recipe_format   # feedrattler's branch
git switch "$B"
F=$(git log --format=%H -1 --grep='^Convert to v1 feedstock')
git switch -c v1-split "$F~1"
git mv recipe/meta.yaml recipe/recipe.yaml
git commit -m "rename recipe file"
git restore --source="$F" --staged --worktree :/   # take the converted tree from feedrattler's commit
git commit -C "$F"                                 # reuse its message
git cherry-pick "$F..$B"                           # replay later commits (e.g. rerender)
git diff --exit-code "$B"                          # must be empty
git branch -f "$B" && git switch "$B" && git branch -D v1-split
git push --force-with-lease origin "$B"
```

Verify with `git log --follow --oneline -- recipe/recipe.yaml` that history continues past the conversion into the `meta.yaml` era.

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
| Constraint version of pkg without depending on them | `requirements.run_constrained` | `requirements.run_constraints` |
| Run exports | `build.run_exports` | `run.run_exports` |
| Ignore run exports by name | `build.ignore_run_exports` | `requirements.ignore_run_exports.by_name` |
| Ignore run exports from package | `build.ignore_run_exports_from` | `requirements.ignore_run_exports.from_package` |
| Homepage | `about.home` | `about.homepage` |
| Package hash | `{{ PKG_HASH }}` | `${{ hash }}` |
| Build number | `{{ PKG_BUILDNUM }}` | No direct equivalent, define `build` or `build_number` in context and use `${{ build }}` |

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
- skipping builds is done by listing skipping conditions under `build.skip`, not with selectors

### Best practices

Old `meta.yaml` recipes often contain something like `{% set name = "..." %}`. The `recipe.yaml` equivalent would be setting `context.name` appropriately.
This should not be done anymore for modern recipes. Instead, inline the name in all used places.
