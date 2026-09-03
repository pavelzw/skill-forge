# go-licenses workaround

If a go-based feedstock fails with `one or more libraries have an incompatible/unknown license`,
this means that `go-licenses` was unable to recover a license from a package.
Often, this license is included in the sources and you can manually copy it.

As an example, the `github.com/alecthomas/chroma/v2` packages uses a `COPYING` file that is not detected.
To fix it:

1. Add `--ignore github.com/alecthomas/chroma/v2` to the `go-licenses` calls
2. Manually copy over the license into the respective folder:

```yaml
- if: win
  then:
    - for /f "delims=" %%i in ('dir /s /b gopath\chroma\COPYING') do mkdir ..\library_licenses\github.com\alecthomas\chroma\v2 & copy "%%i" ..\library_licenses\github.com\alecthomas\chroma\v2\COPYING
  else:
    - find $SRC_DIR/gopath -path "*/chroma/*" -name "COPYING" -exec mkdir -p ../library_licenses/github.com/alecthomas/chroma/v2 \; -exec cp {} ../library_licenses/github.com/alecthomas/chroma/v2/COPYING \; -quit
```
