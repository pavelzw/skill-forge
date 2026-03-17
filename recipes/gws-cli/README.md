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
