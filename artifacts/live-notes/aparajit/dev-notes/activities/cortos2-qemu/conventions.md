# Conventions

## Edit only the toolchain submodule
- Product changes go in `repos/ajit-toolchain` only (cortos2, qemu-ajit source if needed, docs, example scripts).
- Do not edit CoRTOS v1 (`os/rtos/cortos`), aparajit `docker/`, `aparajit-docker`, or root donor qemu for this activity.
- Do not put activity labels (slug, milestone ids) in toolchain code or comments.

## QEMU tree in git
- qemu **source** under `qemu-ajit-v1.0/` may change if an example needs it. Do not commit configure output or `$AJIT_HOME/build/qemu-ajit-v1.0/`.
- `cortos_build/` and `cortos_build_qemu/` stay generated.

## cortos2 qemu module
- New target code in its own module (device map, link, output dir, run helper).
- Default `cortos2 build` path stays C-model. No qemu branches sprinkled through existing C-model copy/build.

## Coverage and skips
- No skip-as-done. Ledger in `docs/cortos2-on-qemu-ajit.md` records pass evidence only after both C-model and qemu exit 0 for that example.

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
- setup: inside container, `source ./set_ajit_home`; existing `./setup.sh`; then `./setup_qemu.sh`. `source` `docker/ajit_build/ajit_env` (`cortos2` on `PATH`; distro `python3` plus `python3-pyelftools`, not the Python 3.6 prefix).
- qemu binary: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` on `PATH` via `ajit_env`.
- test C-model: `cd os/rtos/cortos2/examples/<ex> && ./build.sh && ./run.sh`. `pkill -x ajit_C_system_m` if leftovers.
- test QEMU: `cd os/rtos/cortos2/examples/<ex> && ./build_qemu.sh && ./run_qemu.sh`. Pass = `expected_uart.txt` substrings on stdout, exit 0.
- docs: `docs/cortos2-on-qemu-ajit.md` (coverage + setup). v1 doc: `docs/cortos-on-qemu-ajit.md` (reference only).
