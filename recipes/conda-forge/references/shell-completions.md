# Shell Completions for CLI Packages

Some CLI tools support generating shell completion scripts for bash, zsh, and fish. When a tool supports this, you may want to add these completions so users get tab-completion out of the box, especially when installing via `pixi global`.

When helping a user package a CLI tool, do not add completion steps by default to every recipe template. First check whether the tool supports generating completions. If it does, suggest to the user that the recipe can install them as well, and only add the recipe changes when the user asks for that or it is otherwise clearly in scope.

## Standard Installation Paths

Completion files must be placed in these directories under `$PREFIX/share/`:

| Shell | Path | Naming Convention |
|-------|------|-------------------|
| Bash  | `$PREFIX/share/bash-completion/completions/<name>` | No prefix/suffix |
| Zsh   | `$PREFIX/share/zsh/site-functions/_<name>` | Prefixed with `_` |
| Fish  | `$PREFIX/share/fish/vendor_completions.d/<name>.fish` | Suffixed with `.fish` |

`pixi global` automatically picks up completions from these paths when the tool is exposed under the same name as its binary.

## Adding Completions to a Recipe

### Step 1: Generate completions in the build script

Add the following after your binary is built (Unix only — Windows shells don't use this mechanism):

```yaml
build:
  script:
    # ... your existing build steps ...
    - if: unix
      then:
        - mkdir -p $PREFIX/share/zsh/site-functions $PREFIX/share/bash-completion/completions $PREFIX/share/fish/vendor_completions.d
        - $PREFIX/bin/<name> completion --shell zsh > $PREFIX/share/zsh/site-functions/_<name>
        - $PREFIX/bin/<name> completion --shell bash > $PREFIX/share/bash-completion/completions/<name>
        - $PREFIX/bin/<name> completion --shell fish > $PREFIX/share/fish/vendor_completions.d/<name>.fish
```

Replace `<name>` with the actual binary name throughout.

### Step 2: Test that completions are packaged

Add a `files` section to your `package_contents` test:

```yaml
tests:
  - package_contents:
      bin:
        - <name>
      files:
        - if: unix
          then:
            - share/bash-completion/completions/<name>
            - share/fish/vendor_completions.d/<name>.fish
            - share/zsh/site-functions/_<name>
      strict: true
```

## Detecting the Completion Subcommand

There is no universal standard for how CLI tools expose completion generation. You need to check the tool's `--help` output or documentation. Common patterns:

- `<bin> completion --shell bash`
- `<bin> completion bash`
- `<bin> completions bash`
- `<bin> --generate-completion bash`
- `<bin> generate-completion bash`

When creating a recipe, run `<bin> --help` and `<bin> completion --help` (or similar) to find the correct invocation. If a completion subcommand is available, mention to the user that the recipe can package these files too.

## Cross-Compilation Constraint

Generating completions requires **executing the built binary**. This means it only works when the build platform can run the target binary natively. Cross-compiled builds (e.g., building `osx-arm64` on `osx-64`) cannot run the binary, so the completion generation step will fail.

To handle this, update `conda-forge.yml` to the latest defaults so `osx_arm64` uses native builds instead of cross-compilation:

```yaml
provider:
  osx_arm64: default
```

This switches `osx_arm64` back to the default native build configuration instead of overriding it to use `build_platform` cross-compilation. `linux_aarch64` does not necessarily need this change — cross-compiled ARM binaries can still be executed on `linux_64` via QEMU emulation, so the completion generation steps work there.

If adding completions to an existing feedstock that uses cross-compilation, you will need to:

1. Remove `osx_arm64: osx_64` from `build_platform` (if present)
2. Update `provider: osx_arm64: default`
3. Run `pixi exec conda-smithy rerender --commit=auto` to regenerate CI configuration

## When NOT to Add Completions

- The tool doesn't support generating completions (no `completion` subcommand or equivalent)
- The package is Windows-only
- The package is a library, not a CLI tool

If the tool does not support generating completions, simply omit the completion build steps and the `files` test section. The binary-only recipe with `strict: true` will still work correctly.
