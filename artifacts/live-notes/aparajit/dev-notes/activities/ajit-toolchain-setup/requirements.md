# Requirement Definition

## Host-native Docker images
- kind: end-user-interface
- `repos/ajit-toolchain` Docker scripts build and run images for the **host machine type** (this Mac: `linux/arm64`). No `--platform` cross-build on this Mac.
- Same scripts MAY build `linux/amd64` when the host is amd64. That path is untested here; verify later on an amd64 machine if it is messy.
- Verify images: `ajit_base` and `ajit_build_dev`. Align `ajit_build` / `ajit_tools` Dockerfiles/`build.sh` to the same Ubuntu and arch rules (grep/read: 24.04 when that milestone runs; host-native; no `--platform`; no leftover 16.04 PPA after the Ubuntu bump). Do not bake or test those two in this activity.
- Container user matches host `id -u` / `id -g`. Scripts MUST NOT require Linux `getent group docker` or `chown …:docker` on Darwin.

## Darwin clone plus build volume
- kind: end-user-interface
- Darwin: bind-mount (or otherwise use) the Mac git clone as container `$AJIT_HOME` **source** so the clone stays editable on the host.
- Darwin: named Docker volume for **build** trees (at least `$AJIT_HOME/build`, including Buildroot unpack/output). Case-pair test (`Foo` / `foo`) on that volume. Document the volume name next to `run.sh`. `attach_shell.sh` `-w` is `$AJIT_HOME`.
- If the Mac clone cannot hold two paths that differ only by case, rename in the submodule (localized). Prefer rename over moving the whole clone onto the volume.
- Linux hosts: keep the existing bind-mount of the git tree (no extra volume required).

## Ubuntu 24.04 Docker base
- kind: end-user-interface
- **Activity complete only if** verified `ajit_base` / `ajit_build_dev` are native-arch **Ubuntu 24.04**. Distro `python3`. Drop Ubuntu 16.04 and the Python 3.6 PPA in that end image.
- First: try native `ubuntu:16.04` on arm64 and **fix** what is reasonably fixable (packages, PPA substitutes). That try is not done until `./setup.sh` (2014.08) and CoRTOS `example_{001,050,100,150}` succeed **on that 16.04 image**, or the failure is noted as unrecoverable in `activity.md`. **`example_250` waits for M4.** Image boot / `uname` alone is not evidence the image works. If unrecoverable, go to the Ubuntu 24.04 milestone. Do not fold 24.04 into the first Docker milestone unless 16.04 is abandoned as unrecoverable **and** that note is written. A green 16.04-only toolchain does not finish the activity.

## SPARC32 uClibc toolchain prefix
- kind: software-interface
- Cross toolchain stays SPARC V8, 32-bit, uClibc or uClibc-ng.
- Prefer prefix `sparc-buildroot-linux-uclibc`. If 2025.02 forces another name, update PATH/wrappers and CoRTOS-facing scripts in the submodule in the same Buildroot milestone. Prove 32-bit SPARC V8 (`-dumpmachine` plus `-v` or preprocessor `#define`).

## Buildroot 2025.02.18 tree
- kind: internal-behavior
- New sibling folder `repos/ajit-toolchain/buildroot_src_2025.02/` with extracted `buildroot-2025.02.18/` plus named setup/make wrappers.
- One extract: from aparajit-root `buildroot-2025.02.18.tar.gz` into that folder; add the tree to the submodule. Leave the tarball in place, uncommitted. After the PATH switch, setup MUST NOT open the tarball.
- Do not replace `buildroot_src/` (2014.08) in place. Do not upgrade `buildroot_src_64/`. Switch `setup.sh` / PATH to the new tree only at the Buildroot milestone.
- **Activity complete only if** `setup.sh` / `ajit_env` use the 2025.02.18 tree (not 2014.08).

## CoRTOS check set
- kind: end-user-interface
- After a milestone that has a usable toolchain, in `os/rtos/cortos/examples/` run `./build.sh` then `./run.sh` for: `example_001`, `example_050`, `example_100`, `example_150`. **`example_250` is deferred to the Goal e2e (M4)** — 16.04 C-model hung (queue + bget) even without `-w`/`-d`.
- Each pair must exit 0, or print a named success line recorded in the milestone. **C simulator (C-model) only.** Inside the container, `source` `ajit_env`. Not qemu-ajit, not FPGA. Not `example_sort` / `example_stads` / `misc` / full sweep.
- **Activity complete only if** those five runs (including `example_250`) succeed on the C simulator **after** the Ubuntu 24.04 + Buildroot 2025.02.18 image (not as the sole proof of a 16.04/2014.08 stack).

## AHIR C libs as nested submodule
- kind: internal-behavior
- Nested git submodule `repos/ajit-toolchain/ahir` from `git@github.com:bb-dev-blocks/ahir.git`, pin `0816fb6d533715d89364551c267642df701b391c` (matches `ahir_release/VERSION`). Branch `marshal_updates`. Aparajit also registers the same pin at `repos/ahir`. Container builds still use the nested tree.
- Milestone 1 must rebuild host C libs from that tree for **native arm64** (`libPipeHandlerDebugPthreads` at least; BitVectors / CtestBench SockPipes / functionLibrary as needed for the C model). Do not treat x86 `ahir_release/lib/*.so` as arm64 evidence.

## Change localization
- kind: internal-behavior
- All product edits live in `repos/ajit-toolchain`. Do not change aparajit `docker/`, `aparajit-docker`, or `scripts/docker/`.
- Host Docker install (`install_docker.sh`) is out of scope; Docker Desktop (or equivalent) is assumed on this Mac.
- Native (non-Docker) `setup.sh` on macOS is out of scope.
- Moving TFLite / NuttX / qemu-ajit into this submodule is out of scope (later).
