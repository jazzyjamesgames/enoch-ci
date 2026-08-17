# enoch-ci

CI harness that builds the [Enoch](https://github.com/andyvand/Enoch) fork of the
Chameleon boot loader (Darwin/XNU, GPL v2) on GitHub's macOS runners.

## Why

Chameleon/Enoch is distributed as source. Building it requires Apple's toolchain:
`Make.rules` passes `-arch i386` (Apple Clang only) and the boot stages are 32-bit
Mach-O binaries, so a Linux `gcc`/`ld` cross-build is not viable without a full
Mach-O cross-toolchain.

The alternative — downloading prebuilt boot binaries from a third-party mirror — means
trusting an unverified binary that runs before the OS with full disk access. Building
from source on a clean runner avoids that.

## What the workflow does

For each of `macos-13`, `macos-14`, `macos-15`:

1. Reports OS, arch, available Xcode versions, and clang version.
2. **Probes i386 support** — modern Xcode dropped 32-bit support, so it first checks
   whether the runner can compile, link, and freestanding-link an `-arch i386` binary
   at all. This is the expected failure point.
3. Installs `nasm`, clones Enoch, and attempts `make`.
4. Reports `file` output and SHA-256 for any produced boot stages
   (`boot0`, `boot1h`, `boot`) and uploads them plus the build log as artifacts.

`continue-on-error` is set on the build step so a failure on one runner still yields
diagnostics from the others.

## Target

A Dell Inspiron 580s (i3-550 Clarkdale, H57, Radeon X1300/RV516) running Mac OS X
Lion 10.7.5 on legacy BIOS/MBR, chainloaded from GRUB.

No Apple software is redistributed here — this repository contains only the build
workflow.
