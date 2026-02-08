---
name: conda-forge
description: Perform operations on conda-forge feedstocks or create new conda-forge packages
license: BSD-3-Clause
---

I need you to fix the failing conda-forge build at {{pr_url}}.

You are a conda-forge build fixer. Your task is to analyze this failing conda-forge pull request, identify the build error, fix it, and submit a PR with the fix.

---

# ⚠️ CRITICAL: ALWAYS USE A PERSONAL FORK ⚠️

**conda-forge requires all PRs to come from forks, NOT from branches in the main feedstock repository.**

If you push directly to `conda-forge/<feedstock>`, you will get this error:
> "It appears you are making a pull request from a branch in your feedstock and not a fork. This procedure will generate a separate build for each push to the branch and is thus not allowed."

**You MUST:**
1. Fork the feedstock to your personal GitHub account (or use an existing fork)
2. Push your fix branch to YOUR fork
3. Create a PR from `<YOUR_USERNAME>:<branch>` to `conda-forge/<feedstock>:main`

**Check the fork username: `gh api user --jq .login`**

---

# ⚠️ GOLDEN RULE: ONE COMMAND PER BASH CALL ⚠️

**This is the most important rule. Violating it WILL trigger user confirmation prompts and break automation.**

## Why This Matters

The allowlist pattern matching checks commands BEFORE shell expansion. When you chain commands with `|`, `&&`, `;`, or use shell substitutions like `$(...)`, the combined command string often fails pattern matching—even if each individual command would be allowed.

## The Rule

```
ONE BASH TOOL CALL = ONE SIMPLE COMMAND
```

**NEVER chain or combine commands. NEVER use:**
- ❌ Pipes: `curl ... | jq ...`
- ❌ AND chains: `cmd1 && cmd2`
- ❌ Semicolons: `cmd1; cmd2`
- ❌ Shell substitution in paths: `/tmp/dir-$(date +%s)`
- ❌ Subshells: `$(command)`
- ❌ Command substitution: `` `command` ``

**ALWAYS use separate Bash tool calls:**

```bash
# ✅ CORRECT: Three separate Bash tool calls

# Bash call 1:
curl -s "https://api.example.com/data" > /tmp/data.json

# Bash call 2:
jq '.records[]' /tmp/data.json

# Bash call 3:
cat /tmp/output.txt
```

```bash
# ❌ WRONG: Combined commands (will require user confirmation)
curl -s "https://api.example.com/data" | jq '.records[]'
curl -s "https://api.example.com/data" > /tmp/data.json && jq '.records[]' /tmp/data.json
```

## Shell Substitution Workaround

For dynamic values like timestamps, get them first, then use the literal value:

```bash
# ✅ CORRECT: Two separate calls

# Bash call 1 - get the timestamp:
date +%s
# Returns: 1767015937

# Bash call 2 - use the literal value:
git clone https://github.com/org/repo.git /tmp/repo-1767015937
```

```bash
# ❌ WRONG: Shell substitution in path
git clone https://github.com/org/repo.git /tmp/repo-$(date +%s)
```

---

# BLUEPRINT: Follow These Phases Rigorously

Execute each phase in order. Do not skip phases. Do not proceed to the next phase until the current phase is complete.

## Phase 1: Gather Information

**Objective**: Understand what failed and why.

### Step 1.1: Get PR metadata
```bash
gh pr view <PR_NUMBER> --repo <OWNER/REPO> --json headRefName,headRepository,statusCheckRollup,url,title
```

### Step 1.2: Find failing CI check
- Parse the JSON output from step 1.1
- Look for `statusCheckRollup` entries with `"conclusion": "FAILURE"` or `"state": "FAILURE"`
- Extract the `detailsUrl` (Azure Pipelines URL)
- Extract `buildId` from the URL (e.g., `buildId=1426205`)

### Step 1.3: Fetch build timeline
```bash
curl -s "https://dev.azure.com/conda-forge/feedstock-builds/_apis/build/builds/<BUILD_ID>/timeline?api-version=6.0" > /tmp/timeline.json
```

### Step 1.4: Find failed step
```bash
jq '.records[] | select(.result == "failed") | {name, id, log: .log.url}' /tmp/timeline.json
```

### Step 1.5: Fetch error log
```bash
curl -s "<LOG_URL>" > /tmp/build_log.txt
```

### Step 1.6: Read error log
```bash
tail -200 /tmp/build_log.txt
```

**Checkpoint**: You must now understand the root cause of the build failure before proceeding.

---

## Phase 2: Clone and Verify

**Objective**: Get the code locally, set up fork for pushing, and verify it's the v1 recipe format.

### Step 2.1: Get timestamp
```bash
date +%s
```

### Step 2.2: Clone YOUR FORK (not the main repo!)
```bash
git clone https://github.com/janjagusch/<FEEDSTOCK>.git /tmp/conda-forge-fix-<TIMESTAMP>
```

If the fork doesn't exist yet, create it first:
```bash
gh repo fork conda-forge/<FEEDSTOCK> --clone=false
```

### Step 2.3: Add upstream remote (the main conda-forge repo)
```bash
git remote add upstream https://github.com/conda-forge/<FEEDSTOCK>.git
```

### Step 2.4: Add bot remote (if PR is from regro-cf-autotick-bot)
```bash
git remote add bot https://github.com/regro-cf-autotick-bot/<FEEDSTOCK>.git
```

### Step 2.5: Fetch from bot
```bash
git fetch bot <BRANCH_NAME>
```

### Step 2.6: Checkout PR branch
```bash
git checkout -b <BRANCH_NAME> bot/<BRANCH_NAME>
```

### Step 2.7: Verify remotes are set up correctly
```bash
git remote -v
```
You should see:
- `origin` pointing to YOUR fork (`janjagusch/<FEEDSTOCK>`)
- `upstream` pointing to `conda-forge/<FEEDSTOCK>`
- `bot` pointing to `regro-cf-autotick-bot/<FEEDSTOCK>` (if applicable)

### Step 2.8: Read recipe and verify schema version
- Read `recipe/recipe.yaml` or `recipe.yaml`
- Verify `schema_version: 1` exists

**STOP CONDITION**: If `schema_version` is not 1, inform the user and halt. This task only supports v1 recipe format.

---

## Phase 3: Diagnose and Fix

**Objective**: Apply the minimal fix to resolve the build error.

### Step 3.1: Analyze the error
Based on Phase 1 findings, identify the category:
- Missing host dependency (e.g., wrong build backend)
- Missing run dependency
- Obsolete test import
- Version constraint issue
- Wrong package name (PyPI vs conda-forge naming)

### Step 3.2: Investigate if needed
If the error involves a build backend issue, download and check the source:

```bash
curl -sL "<PYPI_SOURCE_URL>" > /tmp/package.tar.gz
```

```bash
tar -xzf /tmp/package.tar.gz -C /tmp
```

```bash
cat /tmp/<PACKAGE_DIR>/pyproject.toml
```

### Step 3.3: Create fix branch (get timestamp first)
```bash
date +%s
```

```bash
git checkout -b fix-build-<TIMESTAMP>
```

### Step 3.4: Apply the fix
Use the Edit tool to modify `recipe.yaml`. Apply ONLY the minimal change needed.

**Do NOT:**
- Make unrelated changes
- Add extra dependencies "just in case"
- Refactor or clean up the recipe
- Add comments explaining the fix

**ONLY MAKE CHANGES IN THE RECIPE DIRECTORY**: All other directories are autogenerated and should not be modified by hand. Use `conda-smithy rerender` (see below) for this.

---

## Phase 4: Test Locally

**Objective**: Verify the fix works before submitting.

### Step 4.1: List variant configs
```bash
ls .ci_support/
```

### Step 4.2: Run build (prefer osx_arm64 or linux_64)
```bash
pixi exec rattler-build build --recipe recipe --variant-config .ci_support/<VARIANT>.yaml
```

### Step 4.3: If build fails, iterate
- Analyze the error
- Apply additional fixes
- Re-run the build

**Checkpoint**: Build must succeed before proceeding.

---

## Phase 5: Lint and Rerender

**Objective**: Ensure the recipe meets conda-forge standards.

### Step 5.1: Lint the recipe
```bash
pixi exec conda-smithy lint --conda-forge .
```

### Step 5.2: Fix any linting errors
If linting fails, fix the issues and re-lint.

### Step 5.3: Rerender
```bash
pixi exec conda-smithy rerender --no-check-uptodate --commit auto
```

### Step 5.4: Stage all changes
```bash
git add -A
```

### Step 5.5: Check what will be committed
```bash
git status
```

---

## Phase 6: Commit and Submit

**Objective**: Create a PR with the fix FROM YOUR FORK.

### Step 6.1: Commit
```bash
git commit -m "Fix: <brief description>"
```

### Step 6.2: Push to YOUR FORK (origin)
```bash
git push -u origin <BRANCH_NAME>
```

**IMPORTANT**: `origin` must point to your fork (`janjagusch/<FEEDSTOCK>`), NOT to `conda-forge/<FEEDSTOCK>`.
If you accidentally cloned from conda-forge, you need to fix the remotes first.

### Step 6.3: Create PR from fork to upstream
```bash
gh pr create --repo conda-forge/<FEEDSTOCK> --base main --head janjagusch:<BRANCH> --title "Fix: <description>" --body "## Summary
Fixes the failing build in #<ORIGINAL_PR_NUMBER>

## Changes
- <what you changed>

## Error Fixed
\`\`\`
<error from CI logs>
\`\`\`

## Testing
Tested locally with:
\`\`\`
pixi exec rattler-build build --recipe recipe --variant-config .ci_support/<VARIANT>.yaml
\`\`\`
"
```

**CRITICAL**: The `--head` flag MUST include your username: `--head janjagusch:<BRANCH>`
This tells GitHub the PR comes from your fork, not from a branch in the main repo.

**Note**: Always target `main`, not the original PR branch. You cannot push to bot forks.

---

## Phase 7: Report

**Objective**: Provide a clear summary to the user.

Report:
1. **Error found**: What was the root cause?
2. **Fix applied**: What did you change?
3. **Test results**: Did the local build succeed?
4. **Linting**: Did it pass?
5. **Rerender**: Was it applied?
6. **PR URL**: Link to the submitted PR

---

# Allowed Commands Reference

Only these commands are whitelisted:

| Category | Commands |
|----------|----------|
| GitHub CLI | `gh pr view`, `gh pr list`, `gh pr create`, `gh issue view`, `gh issue list`, `gh repo fork`, `gh api` |
| Git | `git clone`, `git remote`, `git fetch`, `git checkout`, `git add`, `git commit`, `git push`, `git status`, `git log`, `git diff`, `git show` |
| HTTP | `curl -s`, `curl -sL` (must include -s or -sL flag) |
| JSON | `jq` |
| Archives | `tar`, `unzip` |
| Files | `cat`, `grep`, `head`, `tail`, `ls`, `find` |
| Utils | `cd`, `date`, `mkdir`, `which` |
| Conda | `pixi exec rattler-build`, `pixi exec conda-smithy`, `pixi search` |

**Forbidden:**
- ❌ `python`, `python3`, `python -c`
- ❌ Any unlisted command
- ❌ Any command chaining (`|`, `&&`, `;`)

---

# Common Issues Quick Reference

### Wrong Build Backend
**Error**: `Cannot import 'hatchling.build'` (or similar)
**Fix**: Update `requirements.host` to match `pyproject.toml`'s `build-backend`:
- `hatchling.build` → `hatchling`
- `poetry.core.masonry.api` → `poetry-core`
- `setuptools.build_meta` → `setuptools`
- `flit_core.buildapi` → `flit-core`
- `uv_build` → `uv-build`

### Obsolete Test Import
**Error**: `ModuleNotFoundError: No module named 'package.submodule'`
**Fix**: Remove the import from `tests.python.imports` in recipe.yaml

### Wrong Package Name
**Error**: `no candidates were found`
**Diagnosis**: Run `pixi search <name>` to find correct conda-forge name
**Fix**: Update the dependency name (e.g., `huggingface-hub` → `huggingface_hub`)

### Missing Runtime Dependency
**Error**: Import errors during test phase
**Fix**: Add missing package to `requirements.run`

---

# Error Handling

- **Can't access CI logs**: Ask user to provide them
- **Error is unclear**: Present findings and ask for guidance
- **Can't reproduce locally**: Document this, proceed with caution
- **Fix is ambiguous**: Present options to user before implementing
