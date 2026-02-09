# Inspect a Conda Package

## Finding the Package

Search for the package in the output directory:

```bash
find output/ -name "*<package-name>*" -type f \( -name "*.conda" -o -name "*.tar.bz2" \) 2>/dev/null
```

## Extracting and Inspecting

### For `.conda` packages (ZIP containing zstd-compressed tarballs)

`.conda` files are ZIP archives containing:
- `metadata.json` - package metadata
- `pkg-<name>-<version>-<build>.tar.zst` - the package files
- `info-<name>-<version>-<build>.tar.zst` - package info/metadata

```bash
# Create a temp dir and extract
TMPDIR=$(mktemp -d)
cd "$TMPDIR"

# Unzip the outer container
unzip -q <package.conda>

# List contents
ls -la

# Extract the package contents
# Option 1 (requires GNU tar built with zstd support):
tar --zstd -xf pkg-*.tar.zst

# Option 2 (more portable: requires zstd and tar):
zstd -d < pkg-*.tar.zst | tar -xvf -

# Extract the info
# Option 1 (requires GNU tar built with zstd support):
tar --zstd -xf info-*.tar.zst

# Option 2 (more portable: requires zstd and tar):
zstd -d < info-*.tar.zst | tar -xvf -
```

### For `.tar.bz2` packages

```bash
TMPDIR=$(mktemp -d)
cd "$TMPDIR"
tar -xjf <package.tar.bz2>
```

## What to Inspect

### 1. Package metadata

```bash
# Package index info
cat info/index.json | jq .

# Package paths (all files in the package)
cat info/paths.json | jq .

# Dependencies
cat info/index.json | jq '.depends'

# License
cat info/licenses/*
```

### 2. File listing

```bash
# List all files
cat info/paths.json | jq -r '.paths[].path'

# List binaries
ls bin/ 2>/dev/null

# List libraries
ls lib/ 2>/dev/null

# List headers
find include/ -type f 2>/dev/null
```

### 3. Binary inspection

```bash
# Check linked libraries (macOS)
otool -L bin/<binary> 2>/dev/null || true
otool -L lib/<library>.dylib 2>/dev/null || true

# Check linked libraries (Linux)
ldd bin/<binary> 2>/dev/null || true
ldd lib/<library>.so 2>/dev/null || true

# Check RPATH (macOS)
otool -l bin/<binary> 2>/dev/null | grep -A2 RPATH || true

# Check RPATH (Linux)
readelf -d bin/<binary> 2>/dev/null | grep -i rpath || true
patchelf --print-rpath bin/<binary> 2>/dev/null || true
```

### 4. Relocatability check

Look for hardcoded paths that should be relative:
```bash
# Search for absolute paths in text files
grep -r "/home\|/usr/local\|/opt" --include="*.pc" --include="*.la" --include="*.cmake" . 2>/dev/null

# Check for placeholder prefix in binaries
grep -rl "placehold_placehold" bin/ lib/ 2>/dev/null
```

### 5. pkg-config files

```bash
cat lib/pkgconfig/*.pc 2>/dev/null
```

## Cleanup

```bash
rm -rf "$TMPDIR"
```

## Summary

Report:
1. Package name, version, and build string
2. Dependencies listed in index.json
3. Key files installed (binaries, libraries, headers)
4. Any issues found (hardcoded paths, missing rpaths, etc.)
5. License information
