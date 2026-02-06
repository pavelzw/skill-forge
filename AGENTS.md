# Project Summary

This repository contains conda packages for coding agent skills.

## Structure

"Skills" are text files that AI-based coding agents can read to enhance their understanding of a complex topic or tool, like an internal library or company-specific guidelines and knowledge. They're usually structured as markdown files with brief documentation, instructions, code examples, and usage guidelines. A skill may reference additional markdown files in a `references/` subdirectory that are read on demand.
Every skill is shipped in a conda package located in `etc/agent-skills/<skill>/SKILL.md` (plus additional files).

## Mirroring from skills.sh

When mirroring a skill from https://skills.sh/, start with the example in `examples/mirror-skills.sh`. You then need to look into the repository and find the actual skill directory.

### Update Strategies for Mirrors

If you build a skill that fetches data from a remote source, please also add it to `.github/workflows/autobump.yml` with appropriate update strategies.

- **github-latest-release**: Use this strategy if the repository has releases and the latest release is somewhat recent.
- **git-main**: Use this strategy otherwise.

## Updating skill contents

When the user asks you to update the content of a skill, follow these steps:
1. Check out the latest version of the tool or API that the skill is based on to a temporary location.
2. Investigate which parts of the skill still accurately reflect the current state of the tool or API and which parts need to be updated or added to the skill.
3. Apply the minimal viable update to the skill's content. This means:
   a) NEVER change information in the skill that is still accurate, exhaustive, correct, and up-to-date.
   b) ONLY change information that is outdated, incorrect, or missing.
   c) Often times, this means edits will be local to a subsection or a few paragraphs, or new subsections will be added.
4. If there is a `PROMPT.md` next to the `recipe.yaml`, read that file and also include it in how you update the skill.

## Building your own skills

If you want to build a skill from scratch, you can follow the `examples/example-skill` example.
Note that you don't specify a `source` field in the recipe.yaml file here but just use `$RECIPE_DIR` when building the package.
If the user creates special instructions, you can create a `PROMPT.md` file in the recipe directory that includes those instructions. *Don't include it in the final package*.

## Testing

To build all new skills, you can run

```
pixi run build-new
```

If you want to build a specific skill, you can run

```
pixi run rattler-build build -r recipes/<skill>
```

Please always run `pixi run pre-commit-run` to check for linting errors.
