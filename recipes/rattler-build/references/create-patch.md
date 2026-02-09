# Create a Patch from Build Directory Changes

## Steps

### 1. Find the work directory

```bash
# Get the latest build directory
WORK_DIR=$(tail -1 output/rattler-build-log.txt)
echo "Work directory: $WORK_DIR"
```

### 2. Verify source info exists

```bash
jq . "$WORK_DIR/.source_info.json"
```

This file tracks the original source files so `create-patch` knows what was modified.

### 3. Preview the patch (dry run)

Always do a dry run first to see what would be included:

```bash
cd "$WORK_DIR"
rattler-build create-patch \
  --directory . \
  --name <patch-name> \
  --dry-run
```

Review the output to make sure:
- Only intended changes are included
- No build artifacts (`.o`, `.so`, `.dylib`) are included
- No generated files are included

### 4. Exclude unwanted files

If the dry run shows unwanted files, use `--exclude`:

```bash
rattler-build create-patch \
  --directory . \
  --name <patch-name> \
  --exclude "*.o,*.so,*.dylib,*.a,*.pyc,__pycache__,build/" \
  --dry-run
```

### 5. Include new files

If you added new files that should be in the patch, use `--add`:

```bash
rattler-build create-patch \
  --directory . \
  --name <patch-name> \
  --add "src/new_file.c,include/new_header.h"
```

### 6. Create the actual patch

Once the dry run looks good:

```bash
cd "$WORK_DIR"
rattler-build create-patch \
  --directory . \
  --name <patch-name> \
  --exclude "*.o,*.so,*.dylib,*.a,*.pyc,__pycache__,build/"
```

This creates `<patch-name>.patch` in the current directory.

### 7. Copy the patch to the recipe directory

```bash
# Find which recipe this build came from
# Check the recipe.yaml or build info to find the recipe path
cp "$WORK_DIR/<patch-name>.patch" <recipe-directory>/
```

### 8. Update the recipe

Add the patch to the recipe's source section:

```yaml
source:
  url: https://example.com/source.tar.gz
  sha256: ...
  patches:
    - <patch-name>.patch
```

### 9. Verify by rebuilding

```bash
rattler-build build --recipe <recipe-directory>/recipe.yaml --keep-build
```

## Tips

- Always use `--dry-run` first to preview what will be in the patch
- Use `--exclude` to filter out build artifacts and generated files
- Use `--add` for new files you created that aren't tracked by the original source
- Use `--overwrite` if a patch with the same name already exists
- The `.source_info.json` file must exist (created during source fetching)
- If `.source_info.json` is missing, rebuild the package first with `--keep-build`
