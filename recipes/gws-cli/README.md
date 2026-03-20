# gws-cli

Agent skills for the [Google Workspace CLI](https://github.com/googleworkspace/cli).

## Generating the recipe

The `recipe.yaml` is auto-generated from the upstream repository's SKILL.md files.
To regenerate it, run from the repository root:

```sh
pixi run generate-gws-cli
```

This fetches skill metadata (names, descriptions, dependencies) from
`raw.githubusercontent.com` and writes `recipe.yaml`.

## Notes

### Package naming

All packages are prefixed with `agent-skill-gws-` for consistency. Skills that
don't already have a `gws-` prefix upstream (e.g. `recipe-*`, `persona-*`) get
one added: `recipe-save-email-attachments` becomes
`agent-skill-gws-recipe-save-email-attachments`. The install paths under
`share/agent-skills/` still use the original upstream directory names, so
relative links between SKILL.md files are not affected.

### Skipped validation

The `agentskills validate` test is intentionally skipped for these packages.
The upstream SKILL.md frontmatter uses JSON-style flow sequences (e.g. `["gws"]`)
which `strictyaml` (used by `skills-ref`) rejects.
