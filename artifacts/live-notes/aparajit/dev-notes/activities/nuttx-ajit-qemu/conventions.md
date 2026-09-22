# Conventions

## Product tree
- Port and scripts live in `repos/ajit-toolchain` (`nuttx` submodule for the board and chip, `scripts/test-nuttx-ajit.sh`, `scripts/run-nuttx-ajit.sh`).
- Do not delete or build `repos/nuttx` for this activity.
- Do not use `aparajit-docker` or root `scripts/docker/build-nuttx.sh` for this activity.
- QEMU source under `repos/ajit-toolchain/qemu-ajit-v1.0/` may change if the board cannot boot. Do not commit `build/` output.
- No activity slug or milestone ids in code, comments, or identifiers.

## Git author
- Author and committer: `bb-dev-blocks <basicblocksdevelopers@gmail.com>`.
- Do not rewrite upstream NuttX author or committer names.
- Do not commit unless the user asks. Do not commit the parent gitlink unless the user asks.
- Do not commit secrets or `build/` trees.

## Process doc
- Write the learning doc at `docs/nuttx-on-ajit-qemu.md`.
- Leave `nuttx-ajit.md` as the old handoff. Do not delete it here.

## Setup, build, test and install notes
- images: existing `ajit_base:1.0` and `ajit_build_dev:1.0` from `ajit-toolchain-setup` (Ubuntu 24.04, Buildroot 2025.02.18, `sparc-linux-gcc` 13.4.0, distro `python3`). Do not run `docker/ajit_base/build.sh` or `docker/ajit_build_dev/build.sh` unless the image is missing.
- container: host `repos/ajit-toolchain` at `/home/ajit/ajit-toolchain`. Darwin volume `ajit-toolchain-build` on `$AJIT_HOME/build`.
- start: `cd repos/ajit-toolchain/docker/ajit_build_dev && ./run.sh` then `./attach_shell.sh`.
- setup: inside the container, `source ./set_ajit_home`, `./setup.sh` if the toolchain is missing, `./setup_qemu.sh`, `source docker/ajit_build/ajit_env`.
- host packages: the image has no `kconfig-tweak`. Inside the running container, as root: `apt-get update && apt-get install -y kconfig-frontends`. `./run.sh` recreates the container from the image, so repeat that install after a recreate. Do not rebuild the image for this package.
- compiler: `sparc-linux-gcc` (`sparc-buildroot-linux-uclibc-gcc`). NuttX builds set `CROSSDEV=sparc-linux-`.
- qemu: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` via `ajit_env`.
- submodules: from `repos/ajit-toolchain`, `git submodule update --init nuttx apps`.
- build: inside the container, `./scripts/test-nuttx-ajit.sh leon` (the three LEON configs) and the `nsh` / `smp` modes of that script.
- manual: inside the container, `./scripts/run-nuttx-ajit.sh nsh` and `./scripts/run-nuttx-ajit.sh smp`. Quit: Ctrl-A then X.
- test: inside the container, `./scripts/test-nuttx-ajit.sh` (LEON builds, `-smp 1` `help`/`hello`, `-smp 2` and `-smp 4` CPU bring-up).
- doc: `docs/nuttx-on-ajit-qemu.md` at the aparajit root.
