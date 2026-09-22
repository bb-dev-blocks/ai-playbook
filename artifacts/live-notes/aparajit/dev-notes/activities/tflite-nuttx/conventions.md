# Conventions

## Product tree
- Board config, tflite linker script, and the four NSH programs live in `repos/ajit-toolchain/nuttx` (branch `ajit_main`).
- Doc and script modes live in `repos/ajit-toolchain`.
- Do not edit `repos/ajit-toolchain/apps` or `repos/nuttx`.
- Do not edit cortos2 example sources. Read their ResNet files only.
- Do not use `aparajit-docker` or root `scripts/docker/build-nuttx.sh`.
- No activity slug or milestone ids in code, comments, or identifiers.

## Image directory
- On `ajit1-qemu:tflite`, the four programs are `/tflite/hello_world`, `/tflite/micro_speech`, `/tflite/person_detection`, `/tflite/resnet50`.
- Inputs are files under `/tflite/inputs/`. Invoke `/tflite/<name> <file>`. `cat /tflite/README.md` is the on-image how-to. Do not rely on bare command names.
- Extra inputs live in `tflite-inputs/` at the toolchain root. That directory is gitignored. The build copies it onto `/tflite/inputs/`.

## nsh and smp stay
- Do not change `ajit1-qemu:nsh`, `ajit1-qemu:smp`, or `boards/sparc/ajit1/ajit1-qemu/scripts/linksparc.ld`.
- `ajit1-qemu:tflite` uses its own 128 MiB link region.

## Assets
- Read blobs from the paths in `requirements.md`. Do not commit a second copy into `nuttx`.
- Generated C arrays and object files stay untracked.
- Missing source: build exits non-zero and prints that path.

## Git
- Do not commit during the activity. Commit only when the user asks at the end.
- New `nuttx` commits: author and committer `bb-dev-blocks <basicblocksdevelopers@gmail.com>`.
- Do not rewrite upstream NuttX authors.
- Do not commit the parent gitlink unless the user asks.
- Do not commit secrets or build trees.

## Document
- Write `repos/ajit-toolchain/docs/tflite-on-nuttx.md`.
- Cover how the port works and how to test it (manual commands and scripted modes).

## Setup, build, test and install notes
- images: existing `ajit_base:1.0` and `ajit_build_dev:1.0`. Do not run `docker/ajit_base/build.sh` or `docker/ajit_build_dev/build.sh` unless the image is missing.
- container: host `repos/ajit-toolchain` at `/home/ajit/ajit-toolchain`.
- start: `cd repos/ajit-toolchain/docker/ajit_build_dev && ./run.sh` then `./attach_shell.sh`.
- setup: inside the container, `source ./set_ajit_home`, `./setup.sh` if the toolchain is missing, `./setup_qemu.sh`, `source docker/ajit_build/ajit_env`.
- host packages: image has no `kconfig-tweak`. As root in the container: `apt-get update && apt-get install -y kconfig-frontends`. Repeat after `./run.sh` recreates the container. Do not rebuild the image for that package.
- compiler: `CROSSDEV=sparc-linux-`.
- qemu: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` via `ajit_env`. Guest flag stays `-m 128M`.
- submodules: from `repos/ajit-toolchain`, `git submodule update --init nuttx apps tflite-micro`.
- build and test: inside the container, `./scripts/test-nuttx-ajit.sh tflite-boot`, `tflite-small`, `tflite-resnet`, and `tflite`. Regression: `./scripts/test-nuttx-ajit.sh nsh`.
- manual: `./scripts/run-nuttx-ajit.sh tflite`. Quit: Ctrl-A then X.
- doc: `docs/tflite-on-nuttx.md` under `repos/ajit-toolchain`.
