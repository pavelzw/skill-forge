# conda-forge Skill structure
Rewrite the entire existing conda-forge skill. The only things you will keep are the example recipes, rattler-build-migration.md, cross-compilation, one per language that we already have. The skill.md needs to be restructured entirely. 
Note the following details and structures: We'll first differentiate between new recipes in the staged recipes repository and existing feedstocks, and only afterwards provide all the context and details about building conda packages in general.

First of all we need to differentiate whether that user is trying to create a new feedstock in the staged-recipes repository or update an existing feedstock.
Check the current working directory:
staged-recipes: create new
xyz-feedstock: update existing

### Staged Recipes: Creating new feedstocks
- create recipes in recipes/yourpackage/recipe.yaml
- If we're building a no-arch Python package, we need to provide the following context flag to rattler-build `--context python_min=3.10`

### Existing feedstock: Updating
- we're working on the recipe/recipe.yaml
- always run `-m .ci_support/<platform>.yaml` That's compatible with your local platform architecture 
- When you conclude your changes, run `pixi exec conda-smithy rerender --commit=auto` and `pixi exec conda-smithy lint --conda-forge .` to make sure your changes are compatible with conda-forge's standards.

Note: If you only find a `meta.yaml` in the feedstock, you need to convert it to the new `recipe.yaml` format. See the [rattler-build-migration.md](references/rattler-build-migration.md) guide for details on how to do that, either using the `feedrattler` tool or manually.
