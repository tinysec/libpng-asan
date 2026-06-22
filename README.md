# libpng-asan

ASAN-instrumented build of **Chrome's libpng** — the PNG decoder Chrome ships — plus ready-to-run libFuzzer harnesses. Rebuilt automatically whenever the revision Chrome uses changes.

![build](https://github.com/tinysec/libpng-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/libpng-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/libpng-asan?label=release)

## Staying current

`chrome.lock` pins the exact libpng revision a specific Chrome stable version
ships. The [`track-chrome`](.github/workflows/track.yml) workflow runs every 6
hours: it resolves the latest Chrome stable, walks Chrome DEPS → PDFium DEPS to
find the libpng revision, and when it has changed it bumps `chrome.lock`, tags
`chrome-<version>`, and triggers [`build`](.github/workflows/build.yml). Each
`chrome-<version>` tag becomes a GitHub release.

## Release artifacts

Each release is published at its `chrome-<version>` tag as one **zip per
platform and build mode** (so every asset has a unique name and the directory
structure is preserved). Each zip contains:

- **ASAN build** (`*-asan-*`): the ASAN **static** lib (`libpng16.a` /
  `libpng16_staticd.lib`) and **dynamic** lib (`libpng16.so.*` /
  `libpng16d.dll` + import lib), the public headers (`png.h`, `pngconf.h`,
  `pnglibconf.h`, `zconf.h`, `zlib.h`), and on Windows the ASAN runtime DLL.
- **libFuzzer build** (`*-libfuzzer-*`): self-contained libFuzzer binaries
  (`libpng_colormap_fuzzer`, `libpng_transformations_fuzzer` with `.exe` on
  Windows; `libpng_read_fuzzer` and `libpng_readapi_fuzzer` are Linux-only —
  those harnesses use the POSIX `nalloc.h` allocator), the `png.dict`
  dictionary, and on Windows the ASAN runtime DLL.

## ASAN runtime model (Windows)

The Windows LLVM toolchain ships **dynamic-only** ASAN. Every Windows library
and binary here imports `__asan_*` from `clang_rt.asan_dynamic-x86_64.dll`,
which is shipped inside each Windows zip. **That DLL must sit beside your
executable / DLL at runtime.** Linux links ASAN statically and needs no extra
runtime.

## Fuzzing on Windows

- **Run the shipped libFuzzer binaries directly.** They are self-contained
  fuzzers: extract `clang_rt.asan_dynamic-x86_64.dll` next to the `.exe`, point
  it at a seed-corpus directory, and run e.g.
  `libpng_colormap_fuzzer.exe corpus/`.
- **AFL++ / WinAFL.** Link the ASAN **static** library into a harness exposing
  a target function over a memory buffer, then point WinAFL at that function.
  The static lib + shipped ASAN DLL is the same combination the libFuzzer
  binaries use.

> These builds provide the ASAN-instrumented library to target. They do not
> bundle the AFL++ / WinAFL runner itself (a separate DynamoRIO-based
> toolchain on Windows).
