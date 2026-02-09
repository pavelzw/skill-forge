# Build a Recipe

## Steps

1. **Locate the recipe**: Find the `recipe.yaml` file. Check these locations in order:
   - Direct path (if it's a path to a file)
   - `<name>/recipe.yaml` (if it's a directory)
   - `recipes/<name>/recipe.yaml`
   - `stage0/<name>/recipe.yaml`
   - `stage1/<name>/recipe.yaml`
   - `stage2/<name>/recipe.yaml`

2. **Read and understand the recipe**: Read the `recipe.yaml` to understand what's being built, what sources are needed, and what dependencies are required.

3. **Check for a variants file**: Look for `variants.yaml` next to the recipe.

4. **Build the recipe**:
   - For recipes in `stage0/` or `stage1/`, prefer using `pixi run build-stage0 <name>` or `pixi run build-stage1 <name>`.
   - For recipes in `recipes/`, use:
     ```bash
     rattler-build build --recipe <path>/recipe.yaml --keep-build
     ```
   - Add `--channel conda-forge` if the recipe has dependencies from conda-forge.
   - Always use `--keep-build` so we can debug if something fails.

5. **If the build fails**:
   - Read the build output carefully to identify the error.
   - Check `output/rattler-build-log.txt` to find the work directory: `tail -1 output/rattler-build-log.txt`
   - Navigate to the work directory and inspect:
     - `conda_build.log` for full build output
     - `conda_build.sh` for the build script
     - `build_env.sh` for environment variables
   - Common issues:
     - **Missing dependencies**: Add them to `requirements.host` or `requirements.build`
     - **Wrong configure flags**: Check the build script
     - **Patch failures**: Check if patches apply cleanly
     - **Linking errors**: Check `$PREFIX/lib` and `$BUILD_PREFIX/lib`
   - Fix the recipe and rebuild.

6. **After successful build**: Report what was built and where the package is located (in `output/<platform>/`).
