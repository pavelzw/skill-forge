# Common Issues Quick Reference

## Wrong Package Name

**Error**: `no candidates were found`

**Diagnosis**: Run `pixi search <name>` to find correct conda-forge name. You can also use globs in pixi search: `pixi search 'lib*'`

**Fix**: Update the dependency name (e.g., `huggingface-hub` -> `huggingface_hub`)

## Missing Runtime Dependency

**Error**: Import errors during test phase

**Fix**: Add missing package to `requirements.run`

## Missing Dependency While Compiling

**Error**: Error during compilation depends on language

**Fix**: Add library to host dependencies

## Command Not Found During Compilation

**Error**: Exit code 127: cmake (or any other tool) not found

**Fix**: Add to build dependencies

## Error Handling

- **Can't access CI logs**: Ask user to provide them
- **Error is unclear**: Present findings and ask for guidance
- **Can't reproduce locally**: Document this, proceed with caution
- **Fix is ambiguous**: Present options to user before implementing
