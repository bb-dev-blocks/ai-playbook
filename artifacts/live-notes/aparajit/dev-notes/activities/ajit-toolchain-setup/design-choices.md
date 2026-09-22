# Design Decisions

## Native host arch, not emulation
- level: design-choice
- chosen: Build images for the host’s native Docker arch (`linux/arm64` on this Mac). Detect host type in `build.sh` / related scripts so an amd64 host can build amd64 images.
- why: Emulation is slow and a different bug surface. User asked native arm64; amd64 is host-matched, not a multi-arch manifest.
- alternatives:
  - `--platform linux/amd64` on Apple Silicon: rejected.
  - Multi-arch manifest / cross-build on this Mac: out of scope.

## Ubuntu 24.04 LTS as ajit_base
- level: design-choice
- chosen: `FROM ubuntu:24.04`. Distro `python3`; no `ppa:jblgf0/python`.
- why: 16.04 + that PPA is x86-oriented; Buildroot 2025.02 wants a current host.
- alternatives:
  - Stay on 16.04 for the first milestone: only if an arm64 16.04 image and packages actually work; otherwise fold the bump immediately.
  - Ubuntu 22.04: also LTS; user picked 24.04.

## Try native 16.04 first, then 24.04
- level: design-choice
- chosen: Milestone 1 attempts native `ubuntu:16.04` on arm64 and fixes recoverable breaks. Unrecoverable → note in `activity.md`, then milestone 2 is the 24.04 upgrade. Do not skip the 16.04 attempt.
- why: User kept the sequential upgrade; 16.04 may still be fixable enough to validate Docker/arch/volume before the distro bump.
- alternatives:
  - Fold 24.04 into milestone 1 with no 16.04 try: rejected after verify-plan apply.

## Darwin named volume vs Linux bind-mount
- level: design-choice
- chosen: Darwin: named Docker volume for the container `$AJIT_HOME` work tree. Linux: keep bind-mount of the git repo (`docker/ajit_build_dev/run.sh` today).
- why: Bind-mounts inherit Mac case-insensitivity; Buildroot/package trees can have case-only filename clashes.
- alternatives:
  - Rename colliding files in every package: whack-a-mole.
  - Case-sensitive APFS volume for the clone: extra host setup; not required if the volume holds the work tree.

## Darwin clone for edit, volume for builds
- level: design-choice
- chosen: Darwin bind-mounts the Mac clone as `$AJIT_HOME` source. Named volume holds **build** output (`$AJIT_HOME/build` and Buildroot unpack/output). Rename in-tree if the clone itself hits a case clash. Linux: bind-mount only.
- why: User wants the clone editable on the Mac. Whole-tree volume (layout A) would hide the git checkout from the host editor. Volume still gives a case-sensitive FS for generated trees.
- alternatives:
  - Entire `$AJIT_HOME` on the named volume; clone not the container work tree: rejected after apply.
  - Volume only, no bind-mount: harder to edit from macOS.
- replaces: Darwin named volume vs Linux bind-mount

## New Buildroot folder, keep 2014.08
- level: major-implementation-detail
- chosen: `repos/ajit-toolchain/buildroot_src_2025.02/buildroot-2025.02.18/` plus wrappers. Vendor the extracted tree in the submodule. Leave `buildroot_src/` and the parent tarball.
- why: User asked a new folder and to start using it without depending on the tar after extract. In-place replace of 2014.08 would mix upgrade with deletion.
- alternatives:
  - Replace `buildroot_src/buildroot-2014.08` in place: rejected.
  - Keep using the tarball at setup time: rejected.

## SPARC32 uClibc(-ng), same prefix
- level: design-choice
- chosen: SPARC V8 32-bit uClibc or uClibc-ng; keep `sparc-buildroot-linux-uclibc` if 2025.02 allows.
- why: CoRTOS and existing scripts use that prefix. glibc/musl is a different activity.
- alternatives:
  - glibc SPARC: larger ABI/script churn.
  - Upgrade `buildroot_src_64` in parallel: not needed for CoRTOS.

## Nested AHIR submodule for arm64 C libs
- level: major-implementation-detail
- chosen: `repos/ajit-toolchain/ahir` as a git submodule of `ajit-toolchain`, pin `0816fb6d533715d89364551c267642df701b391c`. Keep `ahir_release/` as the existing x86 drop until arm64 libs are proven.
- why: C-model needs PipeHandler sources; they are not in `ahir_release`. Nested submodule keeps AHIR its own repo and matches `VERSION`.
- alternatives:
  - Vendor a snapshot into `ahir_release`: mixes binaries and sources; harder to update.
  - Clone only on the host outside the submodule: out of this activity’s edit boundary.

## Aparajit tracks AHIR at repos/ahir
- level: implementation-detail
- chosen: aparajit `.gitmodules` also pins https://github.com/madhavPdesai/ahir at `repos/ahir`, same sha as nested `repos/ajit-toolchain/ahir`. Docker/`install_ahir_c_libs.sh` still use the nested tree (`$AJIT_HOME/ahir`); Darwin bind-mount is only the toolchain clone.
- why: User asked a first-class `repos/` checkout to track AHIR beside `ajit-toolchain`. Two gitlinks, one pin.
- alternatives:
  - Track only via nested submodule + `--recursive`: less visible under `repos/`.
  - Drop nested and mount `repos/ahir` into the container: extra Darwin `run.sh` bind; not done.

## AHIR origin is the bb-dev-blocks fork
- level: implementation-detail
- chosen: `origin` for `repos/ahir` and nested `ahir` is `git@github.com:bb-dev-blocks/ahir.git`. Branch `marshal_updates`. Parent `.gitmodules` `repos/ajit-toolchain` URL is `git@github.com:bb-dev-blocks/ajit-toolchain.git`.
- why: User forked AHIR under `bb-dev-blocks` and asked that personal GitHub logins not appear in remotes, notes, or new commits.
- alternatives:
  - Keep upstream as `origin`: rejected.

## Git author
- level: implementation-detail
- chosen: author/committer `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>` on every repo in this project (local `user.name` / `user.email` only).
- why: User required this identity for all commits we make.
- alternatives:
  - Host global git identity: not used for this project.

## AHIR work branch marshal_updates
- level: implementation-detail
- chosen: Local branch `marshal_updates` in `repos/ahir` and `repos/ajit-toolchain/ahir`, starting at `0816fb6d`. `.gitmodules` `branch = marshal_updates`. Do not stay detached at the pin. Do not develop on `master`.
- why: User asked a named branch so HEAD is not detached and later AHIR commits have a place to go.
- alternatives:
  - Stay detached at the gitlink sha: rejected.
  - Push `marshal_updates` to upstream instead of the `bb-dev-blocks` fork: rejected.

## Ubuntu 24.04 host packages
- level: implementation-detail
- chosen: `FROM ubuntu:24.04`. Distro `python3` + apt `python3-pyelftools` / `python3-yaml` (no PPA, no pip break-system-packages). `openjdk-17-jre-headless`, `libncurses-dev`. Skip `ensure_python36.sh` / PATH prefix when `python3` is already ≥3.6.
- why: 24.04 has Python 3.12; leftover volume `opt/python-3.6` must not shadow it. openjdk-8 is gone from this distro.
- alternatives:
  - Keep Python 3.6 prefix on 24.04: rejected (not a Goal pin).

## Verify only ajit_base and ajit_build_dev
- level: implementation-detail
- chosen: Full build/run/test those two. Script-align `ajit_build` / `ajit_tools` only.
- why: Docs already treat `ajit_build_dev` as the daily image; baking the other two is extra time.
- alternatives:
  - Bake all four: deferred.
