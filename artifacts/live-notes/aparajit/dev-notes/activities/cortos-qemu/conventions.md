# Conventions

## Edit only the toolchain submodule
- Product changes go in `repos/ajit-toolchain` only.
- Do not edit aparajit `docker/`, `aparajit-docker`, `scripts/docker/`, or root `cortos/` copy for this activity.
- Leave aparajit-root `qemu-ajit-v1.0/` as donor; do not keep it in lockstep after the copy.
- Do not put activity labels (slug, milestone ids) in toolchain code or comments.

## QEMU tree in git
- Commit qemu **source** under `qemu-ajit-v1.0/`. Do not commit configure output or `$AJIT_HOME/build/qemu-ajit-v1.0/`.
- `cortos_build/` and `cortos_build_qemu/` stay generated.

## CoRTOS qemu module
- New target code in its own module (device map, link, output dir, run helper).
- Default `cortos build` path stays C-model. No qemu branches sprinkled through existing C-model copy/build.

## Commits at activity end
- Do not commit per milestone. User commits when this activity is done.
- Do not commit the parent submodule pointer unless the user asks.
- Do not commit secrets. Huge `build/` trees stay untracked.

## Git author
- Commits in this project: author and committer `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- cwd: `repos/ajit-toolchain/docker/ajit_base` and `.../ajit_build_dev`. Run `./build.sh` / `./run.sh` from that dir.
- attach: `./attach_shell.sh` → `docker exec -u $(id -nu) -w /home/ajit/ajit-toolchain -it ajit_build_dev /bin/bash`.
- setup: inside container, `source ./set_ajit_home`; existing `./setup.sh`; then `./setup_qemu.sh` (name may match the shipped script). `source` `docker/ajit_build/ajit_env`.
- qemu binary: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` on `PATH` via `ajit_env`.
- test C-model: `os/rtos/cortos/examples/{example_001,example_050,example_100,example_150,example_250}`: `./build.sh && ./run.sh`. `pkill -x ajit_C_system_m` if leftovers.
- test QEMU: `cd os/rtos/cortos/examples/<ex> && ./build_qemu.sh && ./run_qemu.sh`. Pass = `expected_uart.txt` substrings on stdout, exit 0.
- docs: `docs/cortos-on-qemu-ajit.md` (coverage + skip reasons + setup).
