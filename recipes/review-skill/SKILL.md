---
name: review-skill
description: Interactive senior-engineer pull request review workflow. Reads all existing PR comments (including pending review comments) before reviewing, drafts findings grouped by severity into a markdown file with proposed GitHub comments, iterates with the human, and only posts via the gh CLI after explicit approval — with a human-written summary.
---

# Review Skill

Use this skill when the user asks you to review a pull request. The workflow is interactive: you draft, the human refines, and only after explicit approval do you post anything to GitHub.

Follow the four phases below in order. Do not skip ahead.

## Phase 1 — Review the PR as a senior engineer

Before forming any opinions of your own, read **all existing comments** on the PR so you don't duplicate, contradict, or ignore prior discussion. This includes:

- Issue-level conversation comments: `gh api repos/<owner>/<repo>/issues/<number>/comments`
- Inline review comments on the diff: `gh api repos/<owner>/<repo>/pulls/<number>/comments`
- Existing reviews and their bodies: `gh api repos/<owner>/<repo>/pulls/<number>/reviews`
- **Pending review comments** authored by the current user that have not yet been submitted: `gh api repos/<owner>/<repo>/pulls/<number>/comments` includes them, but you can also check `gh pr view <number> --json reviews` and look for the user's pending review state. If a pending review already exists, treat its drafts as prior context — do not silently overwrite them.

Then review the diff (`gh pr diff <number>` and `gh pr view <number> --json files`) as a senior engineer would. Focus on:

- **Logic errors** — incorrect conditions, off-by-one bugs, wrong operators, missing branches.
- **Edge cases** — empty inputs, concurrency, partial failures, unusual but valid states.
- **Maintainability** — surprising abstractions, leaky boundaries, code that will be hard to evolve, missing tests for risky paths.

For each potential issue, articulate **why it matters** (what breaks, for whom, under what conditions) and propose a **concrete fix**, not a vague concern. If you are not sure something is a bug, say so — uncertain findings are fine if labeled.

Skip pure style nits that an autoformatter or linter would catch. Skip "consider renaming X" unless the name is actively misleading.

## Phase 2 — Write findings to a markdown file

Create a markdown file at `.review/<pr-number>.md` (create the directory if needed). Group findings into three sections, in this order:

1. `## Blocking` — must be fixed before merge (correctness bugs, security issues, data loss).
2. `## Important` — should be addressed but not strictly merge-blocking (maintainability, missing tests, edge cases unlikely in practice).
3. `## Nits` — minor suggestions, take-it-or-leave-it.

Within each section, sort by impact (highest first).

Each finding must have:

- A **human-friendly ID** in kebab-case that describes the issue, e.g. `commit-race-condition`, `empty-list-crash`, `missing-timeout-on-http-call`. IDs should be stable across iterations so the human can reference them.
- A concrete **code reference** in the form `path/to/file.ext:LINE` or `path/to/file.ext:START-END` (e.g. `hello-world/git.py:7-8`).
- A short **explanation** of why the issue matters.
- A **proposed GitHub comment** — the exact text you would post inline on that code position. Write it as the actual comment, not a description of one. Use a fenced block so the human can copy/edit it directly.

Use this template per finding:

```markdown
### <id>

- **Location:** `path/to/file.ext:LINE`
- **Why it matters:** <one or two sentences>
- **Proposed comment:**

  > <the literal comment text to post on GitHub, in the voice you want it published in>
```

At the top of the file, include a `## Summary` section with a placeholder:

```markdown
## Summary

<!-- TO BE WRITTEN BY THE HUMAN — this is the top-level review comment posted to GitHub. -->
```

You may add an `## Overarching notes` section above the findings if there are themes that don't belong on a single line of code (e.g. "this PR mixes a refactor with a feature change"). These will be folded into the review body, not posted as inline comments.

## Phase 3 — Iterate with the human

Tell the human the file is ready and wait. They will:

- Edit the markdown file directly (reword comments, drop findings, reclassify severity, add their own).
- Or ask you to revise specific findings ("rewrite `commit-race-condition` to be less prescriptive", "drop the nits", "move X to blocking").

Re-read the file before each round of edits — the human's manual changes are the source of truth. Preserve their wording exactly when they've rewritten a proposed comment; do not "improve" it.

Do **not** post anything to GitHub during this phase. Do not call `gh pr review`, `gh pr comment`, or any `gh api ... POST` for review endpoints yet.

## Phase 4 — Post the review (only after explicit approval)

Wait for **explicit approval** to post — phrases like "go ahead and post", "ship it", "post the review". Ambiguous reactions ("looks good", "nice") are not approval; ask.

Before posting, the human **must write the summary comment** themselves. If the `## Summary` section still contains the placeholder, stop and ask them to fill it in. The summary is the top-level review body on GitHub and must be in their voice.

Append the following footer to the human's summary before posting (do not modify the human's text above it):

```
---
<sub>created with [review-skill](https://github.com/pavelzw/skill-forge/tree/main/recipes/review-skill)</sub>
```

Then post the review with the `gh` CLI, batching all inline comments into a single review (not separate comments) so the human's collaborators get one notification:

```bash
gh api \
  --method POST \
  repos/<owner>/<repo>/pulls/<number>/reviews \
  -f event=COMMENT \
  -f body="$SUMMARY_WITH_FOOTER" \
  -F 'comments=@comments.json'
```

Where `comments.json` is an array of `{path, line, body}` (or `{path, start_line, line, body}` for multi-line) entries built from the approved findings file. Use `side: "RIGHT"` (the default) for comments on the new version of the diff.

Use `event=COMMENT` by default. Only use `REQUEST_CHANGES` if the human explicitly asks to request changes, and `APPROVE` if they explicitly ask to approve.

Overarching notes from Phase 2 should be appended to the review body (above the footer), not posted as inline comments.

After posting, share the review URL from `gh`'s response with the human and stop.

## Notes

- If the PR is on a fork, inline comments still work via the same endpoint — no special handling needed.
- If `gh` posting fails (e.g. line not in diff), report the exact error to the human and ask how to proceed; do not silently retry against a different line.
- Never force-push, close, or merge the PR as part of this workflow.
