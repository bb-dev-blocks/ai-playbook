# repository root -- Dev-Guide

System software for AJIT and APARAJIT: docs, pinned repos under `repos/`, and a custom QEMU tree for AJIT v1 bring-up.

## Notes

- Ajit is a SparcV8 based CPU (a 32 bit machine).
- Vision and terms: `.dev-notes/definition.md`.
- Submodule checkout from repo root: `git submodule update --init --recursive`. NuttX for qemu-ajit lives in `repos/ajit-toolchain` (`nuttx` + `apps`); init with `git -C repos/ajit-toolchain submodule update --init nuttx apps`. How to build and boot: `docs/nuttx-on-ajit-qemu.md`.
- Platform bring-up layout and fork entrypoints: `repos/dev-guide.md`.
- `qemu-ajit-v1.0/` — custom QEMU fork for the AJIT v1 SparcV8 SoC (`ajit1_generic` machine, `AJIT1` CPU). Build/run: `qemu-ajit-v1.0/README.md`.
- `docker/` — native arm64 build image (32-bit SPARC V8 cross-compile + qemu-ajit). Master script: `./aparajit-docker`. Details: `docker/README.md`.
- `docs/` — Aparajit presentation, plans, and talk-video artifacts.

## Artifacts

| Name | Description |
|------|-------------|
| `docs/` | LaTeX presentation, plans, transcripts |
| `docs/m1-ajit-ubuntu16-setup-arm64mac.md` | Replicate native arm64 Ubuntu 16.04 AJIT toolchain + C-model CoRTOS baseline; pins `ajit-toolchain` `22206c994` and `ahir` `0816fb6d` |
| `docs/m3-ajit-ubuntu24-buildroot2025-setup-arm64mac.md` | Replicate Ubuntu 24.04 + Buildroot 2025.02.18 SPARC gcc 13 + CoRTOS `001`/`050`/`100`/`150` on the C-model |
| `repos/` | Toolchain, NuttX, TFLite Micro, and related forks |
| `qemu-ajit-v1.0/` | Custom QEMU for AJIT v1 simulation |
| `docker/` | Docker build env (arm64; SPARC V8 cross-compile + qemu-ajit) |
| `scripts/docker/` | Build/test helpers for `aparajit-docker` |
| `nuttx-ajit.md` | Older NuttX + qemu-ajit handoff |
| `docs/nuttx-on-ajit-qemu.md` | Build and boot NuttX on qemu-ajit (single core and SMP) |
| `tflite-ajit.md` | Older TFLite Micro handoff (qemu-user / planned sparc-elf). How to run on qemu-ajit: `repos/ajit-toolchain/docs/tflite-on-qemu-ajit.md` |
| `cortos/` | CoRTOS (co-operative RTOS) for AJIT SPARC V8; see `cortos/dev-guide.md` |
| `machine/ajit1/` | Canonical AJIT-1 SoC definition (YAML + C header) for NuttX and bare-metal |
| `repos/dev-guide.md` | Submodule and fork layout under `repos/` |
| `.dev-notes/` | Project definition, journal, knowledge |
