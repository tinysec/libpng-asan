# libpng-asan

ASAN-instrumented fuzzer build of **Chrome's libpng** — the PNG decoder Chrome ships. Rebuilt automatically when the revision Chrome uses changes.

![build](https://github.com/tinysec/libpng-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/libpng-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/libpng-asan?label=release)

**Engines:** libFuzzer (Linux + Windows) · AFL++ (Linux) · **Sanitizer:** ASAN · **Symbols:** Windows `.pdb` included

`track.yml` polls Chrome stable **daily** and resolves the libpng SHA Chrome actually ships (Chrome DEPS → PDFium DEPS). Unchanged → nothing happens. Changed → bump + tag `chrome-<version>` → `build.yml` runs → new Release.

> Requires a `TRACK_TOKEN` PAT (`contents: write`) so the tracker's push starts the build.

```bash
git tag v1 && git push origin v1   # manual trigger
```

No harness is committed here — the OSS-Fuzz `libpng_read_fuzzer` is fetched at build time.
