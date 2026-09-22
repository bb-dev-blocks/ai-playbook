# Design Decisions

## NuttX submodules inside ajit-toolchain
- level: design-choice
- chosen: `repos/ajit-toolchain/nuttx` tracks `git@github-bb:bb-dev-blocks/nuttx.git` `master`. `repos/ajit-toolchain/apps` tracks Apache nuttx-apps at `bc0ed23a5dea42e9aa42dfccc372fe9f4654d10d`. Build and run only inside `ajit_build_dev`.
- why: That container bind-mounts the whole toolchain repo. A toolchain-local tree is what can replace `repos/nuttx` later. NSH needs the sibling `apps` tree. No `bb-dev-blocks` apps repo exists.
- alternatives:
  - `aparajit-docker` plus `repos/nuttx`: first build idea. Dropped so the mounted toolchain tree is the one we build.
  - Kernel submodule only: `configure.sh` expects `../apps`.

## Leave repos/nuttx in place
- level: design-choice
- chosen: Do not delete `repos/nuttx` in this activity. Do not build it. Live checkout is a fresh clone of the fork, not a cherry-pick of `e7da518196` or `43bd7a6ac4`.
- why: Removal was described as later. Those two commits are Anshuman-authored and are not on the fork.
- alternatives:
  - Delete `repos/nuttx` now: user said later.
  - Replay the Anshuman commits onto the fork: would put that name on the live history.

## Git identity
- level: design-choice
- chosen: New commits are author and committer `bb-dev-blocks <basicblocksdevelopers@gmail.com>`. Fork history keeps original authors. Live submodule has no Anshuman line because it is a fresh clone.
- why: User barred that name on the latest commits. Rewriting 63436 upstream authors would misattribute Apache NuttX.
- alternatives:
  - Rewrite every fork author to `bb-dev-blocks`: rejected.
  - Rewrite the eight Anshuman commits on `repos/nuttx`: that tree is not the live one.

## ajit1-qemu board on the s698pm pattern
- level: major-implementation-detail
- chosen: Chip `arch/sparc/src/ajit1/`. Board `boards/sparc/ajit1/ajit1-qemu/` with configs `nsh` and `smp`. Linker origin `0x00100000`. UART registers from `machine/ajit1`. Qemu argv matches cortos2, including `-m 128M` and SMP cap 4.
- why: `s698pm-dkit` already has an SMP defconfig. The cortos2 map is what qemu-ajit implements.
- alternatives:
  - Boot `s698pm-dkit` on qemu-ajit: LEON memory map does not match `ajit1_generic`.

## Reuse the toolchain-setup image
- level: design-choice
- chosen: Existing `ajit_base:1.0` and `ajit_build_dev:1.0` from Complete `ajit-toolchain-setup`. Ubuntu 24.04, Buildroot 2025.02.18, `sparc-linux-gcc` 13.4.0, distro `python3`. No image rebuild.
- why: That image already builds cortos2, TFLite, and AHIR. User approved on that basis.
- alternatives:
  - Rebuild `ajit_base` before the first NuttX compile: not required up front.

## Buildroot SPARC gcc
- level: major-implementation-detail
- chosen: `CROSSDEV=sparc-linux-` from `ajit_env` inside `ajit_build_dev`.
- why: That image's compiler is `sparc-buildroot-linux-uclibc-gcc`. NuttX's SPARC default prefix is `sparc-gaisler-elf-`, which is not installed there.
- alternatives:
  - Install a Gaisler BCC prefix in the image: extra toolchain, not required for the first boot.

## Scripted checks plus a manual terminal
- level: design-choice
- chosen: `scripts/test-nuttx-ajit.sh` drives UART stdin for `help`, `hello`, and CPU bring-up lines. `scripts/run-nuttx-ajit.sh` leaves stdin on the terminal for the same images.
- why: Repeatable evidence and a hands-on NSH session are both required.
- alternatives:
  - Manual-only: a later session cannot re-prove the goal unattended.
