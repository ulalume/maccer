# maccer (mac68k) — native multi-platform builds

Cross-platform native builds of **maccer**, the assembler macro pre-processor used
by [SGDK](https://github.com/Stephane-D/SGDK) to process `.asm` files. SGDK ships
it as a prebuilt Windows binary named `mac68k.exe`; this repository provides the
source and CI that produce **native binaries for Linux / macOS / Windows**, so a
fully native (no-Wine) SGDK toolchain can support `.asm` projects.

> **Why this exists.** SGDK bundles `mac68k.exe` (Win32 only) with no source in its
> repository, so it cannot be built natively. The source was published separately
> on [gendev.spritesmind.net](http://gendev.spritesmind.net/page-macX.html). This
> repo vendors that GPLv2 source and builds it for every platform.

## Attribution & license

- **maccer** — Copyright **Michael Hope, 1998–1999**. Original macro pre-processor
  (designed for GBDK / ASxxxx).
- **Megadrive/68000 port (XGCC/gcc + comment support)** — **Kaneda, 2005**.
- Licensed under the **GNU General Public License, version 2 (or later)**.

The full license is in [`COPYING`](COPYING); the original author's notes are
preserved in [`README`](README) and [`README_KMod`](README_KMod). The source is
[`main.c`](main.c) (Kaneda's renamed `maccer.c`).

GPLv2 is fully compatible with redistributing compiled binaries: every release
ships the binary together with `COPYING`, `README`, `README_KMod`, `ChangeLog`
and the corresponding source is this repository.

> maccer runs as a standalone build-time pre-processor (it transforms `.asm` text
> into GNU `as` syntax). It is **not linked into** the games it helps build, so its
> GPL does not affect SGDK or user projects — the same way GCC's GPL does not make
> compiled programs GPL.

## Binary name: `mac68k`

SGDK's `common.mk` invokes the tool as `mac68k` (`MACCER := mac68k`). The CI
therefore names the output `mac68k` (`mac68k.exe` on Windows), **not** `maccer`.

## Building locally

`main.c` requires two `-D` defines (the bundled `Makefile` is the original and
references the old `maccer.c` filename, so prefer this one-liner):

```sh
cc -O2 -Wall \
  -DVERSION=0.26 -DVERSION_STRING='"0.26"' -DKMOD_VERSION='"0.2"' \
  -o mac68k main.c -lm
```

## CI / releases

`.github/workflows/build.yml` builds five targets:

| Target | Runner | Notes |
|---|---|---|
| linux-x86_64   | `ubuntu-latest`     | native |
| linux-arm64    | `ubuntu-24.04-arm`  | native |
| macos-arm64    | `macos-15`          | native, linker ad-hoc signed |
| macos-x86_64   | `macos-15`          | `-arch x86_64` cross-build (no Intel runner needed) |
| windows-x86_64 | `ubuntu-latest`     | `x86_64-w64-mingw32-gcc` cross-build |

- **Push / PR** → builds and uploads workflow artifacts (CI check).
- **Tag push `v*`** → builds and **attaches assets to the GitHub Release** for that tag.

To cut a release:

```sh
git tag v0.26-1
git push origin v0.26-1
```

Each asset is a `mac68k-<target>.tar.gz` (`.zip` for Windows) containing the
binary plus the license/readme files.
