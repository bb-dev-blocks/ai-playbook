# Journal

## Pause (Active → Paused)

- why: 16.04 C-model needs AHIR C libs; in-tree `ahir_release` is x86-64 `.so` only. User will fetch source.
- done:
  - native arm64 `ubuntu:16.04` `ajit_base` / `ajit_build_dev`; Darwin volume `ajit-toolchain-build`
  - SPARC gcc on PATH (`sparc-buildroot-linux-uclibc` / `sparc-linux-*` links)
  - public AHIR: https://github.com/madhavPdesai/ahir @ `0816fb6d533715d89364551c267642df701b391c` (`v2/pipeHandler` → `libPipeHandlerDebugPthreads`)
- next: `resume-work` after AHIR C sources are in tree; rebuild arm64 PipeHandler/BitVectors/CtestBench; then antlr3c + CoRTOS five
## Resume (Paused → Active)

- done: public AHIR located; pin `0816fb6d533715d89364551c267642df701b391c`
- next: nested submodule `repos/ajit-toolchain/ahir`; arm64 C libs as M1 test; then antlr3c + CoRTOS five
- watch: keep vendored x86 `ahir_release/*.so` until arm64 rebuild; do not commit experimental `functionLibrary` .so overlays

## Pause (Active → Paused)

- why: M1 16.04 baseline is enough; stop before Ubuntu 24.04
- done:
  - native arm64 16.04 Docker + SPARC gcc + arm64 AHIR/antlr3c/C-model
  - CoRTOS `001`/`050`/`100`/`150` on C-model; `250` deferred to M4; notes in `artifacts/ubuntu16-setup.md`
- next: `resume-work` then M2 (`FROM ubuntu:24.04`, distro `python3`)
- watch: do not keep Python 3.6 or `-w`/`-d` as Goal pins; 2014.08 may fail on 24.04 gcc

## Resume (Paused → Active)

- done: M1 16.04 native arm64 Docker + SPARC gcc + arm64 AHIR/antlr3c/C-model; CoRTOS `001`/`050`/`100`/`150`; `250` deferred to M4
- next: M2 `FROM ubuntu:24.04`; distro `python3`; drop 3.6 PPA; align `ajit_build` / `ajit_tools` Dockerfiles; try 2014.08 `setup.sh` and record fail
- watch: Python 3.6 is not a Goal pin; 2014.08 may fail on 24.04 gcc — skip CoRTOS until 2025.02 tree builds; `example_250` is M4 only

## Pause (Active → Paused)

- why: M3 committed with setup/run docs; stop before `example_250`
- done:
  - Ubuntu 24.04 images + Buildroot 2025.02.18 SPARC gcc 13; CoRTOS `001`/`050`/`100`/`150` on C-model
  - docs: `docs/m3-ajit-ubuntu24-buildroot2025-setup-arm64mac.md` (+ submodule copy)
  - git: `repos/ajit-toolchain` `752c624c3`; aparajit `06c87e3` pins gitlink
- next: `resume-work` then M4 (`example_250`)
- watch: do not commit arm64 `ahir_release` overlays or parent `buildroot-2025.02.18.tar.gz`; do not push until asked

## Resume (Paused → Active)

- done: M3 24.04 + Buildroot 2025 gcc 13; CoRTOS `001`/`050`/`100`/`150`; docs + commits `752c624c3` / `06c87e3`
- next: M4 `example_250` on this stack, then Goal e2e of all five
- watch: do not commit arm64 `ahir_release` overlays or parent tarball; C-model hung on 250 (queue + bget) in M1

## AJIT arm64 24.04 Buildroot 2025 CoRTOS C-model (Active → Complete)

Shipped native `ajit_base`/`ajit_build_dev` on Ubuntu 24.04, SPARC gcc 13.4.0 from vendored Buildroot 2025.02.18, and CoRTOS `001`/`050`/`100`/`150`/`250` on `ajit_C_system_model`. 16.04 + 2014.08 was a working baseline then left as a stepping stone.

Paths: `repos/ajit-toolchain` Docker scripts (no `--platform`; Darwin clone + volume `ajit-toolchain-build`); `buildroot_src_2025.02/`; `pt_load_sections.py` vaddr overlap so mmap includes `.data`; CoRTOS python3/gas/inline/`-fno-pic`; `examples/example_250/main.c` waits on `queueId` so the reader does not `ldstub`-spin during UART. Docs: `docs/m1-…`, `docs/m3-…` (M3 still says skip 250). Pins: toolchain `7702c76ae`, parent `aae3000`; AHIR `0816fb6d`. Author `bb-dev-blocks`.

Decisions that stuck: host-native arch; bind-mount clone + volume for `$AJIT_HOME/build`; keep 2014.08 tree; nested AHIR; do not commit arm64 `ahir_release` or the parent tarball.

Gaps accepted: no push; M3 doc not updated for 250; `ajit_build`/`ajit_tools` not baked; amd64 untested; `ajit_debug_monitor` scons fails; leftover C-model and `-w`/`-d` still easy to misuse.

