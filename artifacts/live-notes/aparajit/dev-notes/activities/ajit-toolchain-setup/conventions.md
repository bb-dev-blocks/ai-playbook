# Conventions

## Edit only the toolchain submodule
- Product changes go in `repos/ajit-toolchain` only.
- Do not edit aparajit `docker/`, `aparajit-docker`, or `scripts/docker/` for this activity.
- Do not put activity labels (slug, milestone ids) in toolchain code or comments.

## Commits after a verified milestone
- After a milestone’s evidence commands pass, commit **inside** `repos/ajit-toolchain` before starting the next milestone.
- Do not commit the parent submodule pointer unless the user asks.
- Do not commit `buildroot-2025.02.18.tar.gz` at the aparajit root (leave it; setup must not need it after extract).
- Do not commit secrets. Huge generated `build/` / Docker layer caches stay untracked.

## Nested AHIR repo
- Path: `repos/ajit-toolchain/ahir`. Origin `git@github.com:bb-dev-blocks/ahir.git`. Work on `marshal_updates` at pin `0816fb6d533715d89364551c267642df701b391c`. Aparajit `repos/ahir` uses the same remote, branch, and sha.
- After `git submodule update`, checkout `marshal_updates` again (update checks out the recorded sha detached).
- Do not rewrite AHIR history. Push AHIR work to the `bb-dev-blocks` fork only. Build products stay uncommitted (`*.so` already gitignored except `ahir_release/**/*.so`). Do not commit rebuilt arm64 overlays into `ahir_release/` (keep the x86 drop; `install_ahir_c_libs.sh` rebuilds host arch).
- Do not replace `ahir_release/` until arm64 libs pass `file` as ARM aarch64.

## Git author
- Commits in this project: author and committer `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.
- Submodule URLs under `bb-dev-blocks` (not a personal GitHub login).

## Docker scripts
- Image arch follows the host (`uname -m` / `docker` default). No `--platform` unless a later sibling says so.
- Darwin `run.sh`: bind-mount the git clone as `$AJIT_HOME`; named volume for `$AJIT_HOME/build` (Buildroot unpack/output). Linux: bind-mount git tree only.
- Mac `build.sh` must not assume Linux `getent group docker`; map container user to host uid/gid. No `chown …:docker` on Darwin.
- If the clone hits a case-only name clash, rename in the submodule.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. Container `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- cwd: `repos/ajit-toolchain/docker/ajit_base` and `.../ajit_build_dev`. Scripts that check `pwd` require `./build.sh` / `./run.sh` from that dir (not via path).
- setup: Docker Desktop on this Mac. Submodule root: `source ./set_ajit_home`. Images: `./build.sh` then `./run.sh` in `docker/ajit_base` then `docker/ajit_build_dev`. `ajit_base` is Ubuntu 24.04; distro `python3`.
- attach: `./attach_shell.sh` → `docker exec -u $(id -nu) -w /home/ajit/ajit-toolchain -it ajit_build_dev /bin/bash`.
- build: inside container, `source ./set_ajit_home`; `./setup.sh` (2014.08 until Buildroot milestone; then `buildroot_src_2025.02` wrappers). Output under `$AJIT_HOME/build` (Darwin volume `ajit-toolchain-build`).
- test: Goal e2e CoRTOS: `os/rtos/cortos/examples/{example_001,example_050,example_100,example_150,example_250}`: `./build.sh` then `./run.sh`; both exit 0. C-model `run_cmodel.sh.tpl` omits `-w`/`-d`; `-d` waits for gdb. Stop leftovers with `pkill -x ajit_C_system_m`.
- install: no extra prefix; work tree + `ajit_env` PATH.
- M3 extract: `tar` aparajit-root `buildroot-2025.02.18.tar.gz` into `repos/ajit-toolchain/buildroot_src_2025.02/buildroot-2025.02.18/`.
