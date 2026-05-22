---
name: pr-review
description: Interactive PR review workflow. Reads existing PR comments (including pending review drafts) before reviewing, drafts severity-grouped findings into a markdown file, iterates with the human, and only after explicit approval pushes them as a pending review on GitHub. Never submits the review — the human writes the summary and submits in the UI.
---

# PR Review

Use this skill when the user asks you to review a pull request. The workflow is interactive: you draft, the human refines, and only after explicit approval do you push the inline comments to GitHub **as a pending review**. You never submit the review — the human adds the summary and submits in the GitHub UI.

Follow the four phases in order.

## Phase 1 — Review the PR as a senior engineer

Before forming opinions, read **all existing comments** so you don't duplicate, contradict, or ignore prior discussion:

- Conversation and inline review comments: `gh pr view <number> --comments`
- Existing reviews: `gh api repos/<owner>/<repo>/pulls/<number>/reviews`
- **Pending review drafts** authored by the current user: `gh pr view <number> --json reviews`. If a pending review exists, treat its drafts as prior context — do not silently overwrite them.

Then review the diff (`gh pr diff <number>`) as a senior engineer would. Focus on:

- **Logic errors** — wrong conditions, off-by-one, wrong operators, missing branches.
- **Edge cases** — empty inputs, concurrency, partial failures, unusual but valid states.
- **Maintainability** — leaky abstractions, code that will be hard to evolve, missing tests for risky paths.

For each issue, articulate **why it matters** and propose a **concrete fix**, not a vague concern. Label uncertain findings as such. Skip style nits that a formatter or linter would catch.

## Phase 2 — Write findings to a markdown file

Create `.review/<pr-number>.md`. Group findings into three sections, sorted by impact within each:

1. `## Blocking` — must be fixed before merge (correctness, security, data loss).
2. `## Important` — should be addressed but not merge-blocking.
3. `## Nits` — take-it-or-leave-it.

Each finding needs a stable kebab-case ID, as short as possible while still being unambiguous (e.g. `commit-race`, not `possible-race-condition-on-commit`), that the human can reference across iterations.

Template per finding:

```markdown
### <id>
`path/to/file.ext:LINE` — <why it matters, one or two sentences>

> <literal comment text to post on GitHub>
```

You may add an `## Overarching notes` section above the findings for themes that don't belong on a single line of code. Surface these to the human when handing off — they belong in the human's summary, not on individual lines.

## Phase 3 — Iterate with the human

Tell the human the file is ready and wait. They will edit the file directly or ask you to revise specific findings. Re-read the file before each round — the human's edits are the source of truth, and you must preserve their wording exactly. Do not "improve" rewritten comments.

Do **not** post anything to GitHub during this phase.

## Phase 4 — Push the inline comments as a pending review (only after explicit approval)

Wait for **explicit approval** — phrases like "go ahead and push", "ship it". Ambiguous reactions ("looks good") are not approval; ask.

You will create a **pending review** containing only the inline comments. You do **not** submit it. Do not pass `event=APPROVE`, `event=COMMENT`, or `event=REQUEST_CHANGES`. The human will write the summary and submit the review themselves in the GitHub UI.

Omit both `event` and `body` so the review stays in pending state with no top-level summary:

```bash
gh api \
  --method POST \
  repos/<owner>/<repo>/pulls/<number>/reviews \
  -F 'comments=@comments.json'
```

`comments.json` is an array of `{path, line, body}` (or `{path, start_line, line, body}` for multi-line) built from the approved findings.

After pushing, tell the human the inline comments are pending on the PR and ask them to write the summary and submit in the GitHub UI. Then stop.

## Notes

- If pushing fails (e.g. line not in diff), report the exact error and ask how to proceed; do not silently retry against a different line.
- Never submit, approve, comment-publish, request-changes, force-push, close, or merge the PR as part of this workflow.
