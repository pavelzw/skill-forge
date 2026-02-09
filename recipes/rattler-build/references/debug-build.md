# Debug a Failed Build

## Debugging Workflow

### 1. Find the work directory

```bash
# Get the latest build directory
tail -1 output/rattler-build-log.txt
```

### 2. Inspect the build log

Read `conda_build.log` in the work directory for the complete build output. This contains everything with path replacements applied ($PREFIX, $BUILD_PREFIX, etc.).

Look for:
- Compilation errors (missing headers, undefined symbols)
- Linking errors (missing libraries, undefined references)
- Configure failures (missing dependencies, wrong flags)
- Test failures
- File-not-found errors

### 3. Examine the build environment

Read `build_env.sh` to understand what environment variables were set. Key variables:
- `$PREFIX` - Host prefix (where packages install to)
- `$BUILD_PREFIX` - Build tools prefix
- `$SRC_DIR` - Source directory
- `$RECIPE_DIR` - Recipe directory

### 4. Check source info

```bash
cat .source_info.json | jq .
```

This shows what sources were downloaded and where they were extracted.

### 5. Investigate the issue

Common failure patterns:

**Missing header files**:
- Check if the dependency is listed in `requirements.host`
- Search for the header in `$PREFIX/include/`

**Undefined symbols / linking errors**:
- Check if the library is in `requirements.host`
- Look in `$PREFIX/lib/` for the library
- Check if `-L$PREFIX/lib` is in the linker flags

**Configure script failures**:
- Check the configure script flags in `conda_build.sh`
- Try running configure manually with verbose output

**Patch failures**:
- Check if the patch applies cleanly to the current source version
- Use `rattler-build create-patch` to create updated patches

**Relocatability issues**:
- rattler-build patches rpaths using `$ORIGIN` / `@loader_path`
- Binary placeholder string replacement on install
- Check for hardcoded paths in binaries or text files

### 6. Test fixes interactively

```bash
cd <work-directory>
source build_env.sh
# Now you have the full build environment
# Try running commands manually:
bash -x conda_build.sh 2>&1 | less
```

### 7. Create a patch if needed

If you modified source files to fix the issue:

```bash
cd <work-directory>
rattler-build create-patch --directory . --name <fix-name> --dry-run
# If it looks good:
rattler-build create-patch --directory . --name <fix-name>
```

### 8. Update the recipe

Apply fixes to the `recipe.yaml`:
- Add missing dependencies
- Fix build script commands
- Add patches
- Fix configure flags

### 9. Rebuild

```bash
rattler-build build --recipe <path>/recipe.yaml --keep-build
```

## Tips

- Always use `--keep-build` when debugging to preserve the work directory
- Use `bash -x conda_build.sh` to see exactly what commands are executed
- Check both `$PREFIX` and `$BUILD_PREFIX` for dependencies
- The `conda_build.log` has the full build output with path replacements
- `rattler-build debug-shell` can drop you into a shell with the build environment set up
