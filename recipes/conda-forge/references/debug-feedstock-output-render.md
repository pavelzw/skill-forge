# Debug feedstock output rendering

Use this skill when staged-recipes builds pass, but feedstock creation, rerendering, or output/package-name discovery fails while rendering a variant.

## Key distinction

A successful staged-recipes build does not guarantee that the recipe can render against every generated `.ci_support` variant. The admin-requests feedstock creation flow initializes a feedstock, runs `conda-smithy rerender`, then discovers/registers output package names via `conda-forge-feedstock-ops`. That parser can render variants (including platforms where outputs are skipped) that the normal staged build did not render.

Rattler evaluates top-level `context` expressions before output-level `build.skip` can remove an output. Therefore, a variable used in context must be present or safely guarded even if the corresponding package is unsupported on that platform.

## Reproduce with GitHub Actions

Use a disposable repo with Actions enabled. Check out `conda-forge/staged-recipes` and `conda-forge/admin-requests`, configure a Git author before `conda-smithy init` (which commits the initial feedstock), and use the admin-requests environment. The relevant sequence is:

```yaml
      - name: Configure Git identity
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"

      - name: Check out staged-recipes
        uses: actions/checkout@v4
        with:
          repository: conda-forge/staged-recipes
          path: staged-recipes

      - name: Check out admin-requests
        uses: actions/checkout@v4
        with:
          repository: conda-forge/admin-requests
          path: admin-requests

      - name: Set up admin-requests environment
        uses: mamba-org/setup-micromamba@v2
        with:
          environment-file: admin-requests/environment.yml
          cache-environment: true

      - name: Initialize and rerender temporary feedstock
        shell: bash -el {0}
        env:
          FEEDSTOCK_DIR: ${{ runner.temp }}/debug-feedstock
        run: |
          set -euxo pipefail
          conda-smithy init staged-recipes/recipes/<recipe> \
            --feedstock-directory "$FEEDSTOCK_DIR"
          conda-smithy rerender --no-check-uptodate \
            --feedstock_directory "$FEEDSTOCK_DIR"

      - name: Install output parser
        shell: bash -el {0}
        run: micromamba install -n cf -c conda-forge conda-forge-feedstock-ops

      - name: Print generated variants
        env:
          FEEDSTOCK_DIR: ${{ runner.temp }}/debug-feedstock
        run: |
          for variant in "$FEEDSTOCK_DIR"/.ci_support/*.yaml; do
            echo "--- $variant"
            cat "$variant"
          done

      - name: Parse outputs
        shell: bash -el {0}
        env:
          FEEDSTOCK_DIR: ${{ runner.temp }}/debug-feedstock
        run: |
          python -c '
          from conda_forge_feedstock_ops.parse_package_and_feedstock_names import parse_package_and_feedstock_names
          print(parse_package_and_feedstock_names("'"$FEEDSTOCK_DIR"'", use_container=False))
          '

      - name: Upload generated variants
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: generated-feedstock-variants
          path: |
            ${{ runner.temp }}/debug-feedstock/.ci_support/
            ${{ runner.temp }}/debug-feedstock/conda-forge.yml
          if-no-files-found: warn
          include-hidden-files: true
```

The separate parser install is needed if the chosen environment file does not include `conda-forge-feedstock-ops`. Upload hidden `.ci_support` files with `include-hidden-files: true`; print them before the failing parser step too, so variants are visible in logs even if artifact upload fails.

## Local one-variant render

Download/extract the generated `.ci_support` artifact and run from the staged-recipes root. With pixi's `build` environment:

```powershell
pixi run -e build rattler-build build --render-only --recipe recipes\<recipe> --target-platform win-64 --build-platform win-64 -m "$env:TEMP\debug-feedstock\.ci_support\win_64_.yaml"
```

Change platform and variant path to the failing configuration. This renders only; it does not build or upload a package.

## Debugging checklist

1. Confirm which stage failed: `conda-smithy init`, rerender, parser import, or parser's rattler render. Read the traceback and identify the exact render command/target platform.
2. Inspect the exact generated YAML passed to the renderer; compare it with successful platform variants. Do not assume system pinning supplied every variable.
3. Remember generated variants can be reduced to variables needed by build outputs. A variable absent from a Windows `.ci_support` file may be expected if output distributions are skipped there.
4. For optional context values, guard undefined names before reading them. Example:

   ```yaml
   context:
     cuda: ${{ cuda_compiler_version is defined and cuda_compiler_version != "None" }}
     string_prefix: ${{ "cuda" ~ (cuda_compiler_version | replace(".", "")) if cuda else "cpu" }}
     abi3: ${{ is_abi3 is defined and is_abi3 }}
   ```

   Then consistently use the guarded `cuda`/`abi3` values in all expressions that would otherwise read the optional variables, including `skip`, conditional requirements, and tests. Guard the first reference; replacing only later expressions will not help.
5. Re-run render-only for the failing platform after each fix. It may report the next missing variable; repeat until rendering completes, then rerun the output parser.
6. Keep platform exclusions and package behavior intentional: a context fallback prevents render errors but must not accidentally enable unsupported outputs or alter supported platform variants.

## Example: optional CUDA and ABI3 variants

If CUDA compiler pinning is unavailable on an unsupported platform, a direct top-level comparison such as `${{ cuda_compiler_version != "None" }}` errors because the name is undefined. Define a guarded context alias and use it elsewhere:

```yaml
context:
  cuda: ${{ cuda_compiler_version is defined and cuda_compiler_version != "None" }}
  abi3: ${{ is_abi3 is defined and is_abi3 }}
```

Likewise, use `abi3` instead of direct `is_abi3` references throughout. Retain platform `skip` rules; the aliases only make metadata rendering robust and should preserve values when variables are defined.
