# Design Decisions

## Vendor qemu-ajit as a source copy in the toolchain
- level: design-choice
- chosen: Copy `qemu-ajit-v1.0/` into `repos/ajit-toolchain/qemu-ajit-v1.0/` (source only; omit `build/`). One-shot for this activity. Do not dual-maintain aparajit-root qemu or `aparajit-docker`.
- why: Product home is the toolchain container story; examples already live in that submodule.
- alternatives:
  - Submodule of aparajit qemu path: extra pin dance; user asked for a copy.
  - Keep qemu only at aparajit root: fights “run CoRTOS in ajit_build_dev”.

## Two builds, same sources
- level: design-choice
- chosen: Separate qemu image (`cortos_build_qemu/`) with qemu RAM/UART. C-model build/run scripts unchanged.
- why: Layout mismatch (RAM `0x0` + FPGA UART vs RAM `0x00100000` + qemu UART). One binary cannot honestly serve both without retargeting qemu to the FPGA map.
- alternatives:
  - Teach qemu the FPGA/C-model map: bigger qemu machine change; deferred.

## QEMU link at 0x00100000
- level: major-implementation-detail
- chosen: qemu target links at `0x00100000` to match qemu-ajit RAM. Do not apply C-model `--ramstart` 16MB-alignment to this target.
- why: Matches `ajit1_generic` today.
- alternatives:
  - Move qemu RAM to `0x01000000` and reuse `--ramstart`: later; note in docs.
- later: consider 16MB-aligned qemu RAM so both targets share one rule.

## QEMU skips CoRTOS MMU enable
- level: major-implementation-detail
- chosen: qemu target leaves MMU off after boot (physical map at `0x00100000`). C-model still enables MMU.
- why: qemu-ajit PROM occupies `0`..`1MiB`; CoRTOS C-model page-table stub maps `0x0`. Enabling that MMU after a `0x00100000` link would unmap the image.
- alternatives:
  - Full SRMMU tables for qemu RAM: later if an example needs MMU.

## QEMU ELF entry is `_start`
- level: implementation-detail
- chosen: after qemu link, `sparc-linux-objcopy --set-start` to `_start`. Leave `compileToSparcUclibc.py` `-e main` for C-model.
- why: qemu `-kernel` jumps to `e_entry`; `-e main` skipped CoRTOS init (FPU trap).
- alternatives:
  - Change `-e` in compileToSparc for everyone: would alter C-model tooling.

## qemu asr29 matches CoRTOS thread id
- level: major-implementation-detail
- chosen: `helper_rdasr29` returns `0x50520000 | (core << 8) | thread`.
- why: CoRTOS `_start` compares `%asr29` to `0x50520000`. Old qemu returned `0` for CPU0, so init jumped to `CORTOS_HALT_ERROR` / `ta 0` (trap 0x80) before `main`.
- alternatives:
  - Mask asr29 in CoRTOS: would diverge from the C-model ABI.

## `cortos build --target qemu`
- level: design-choice
- chosen: CLI target flag; one `config.yaml` per example. Thin `build_qemu.sh` / `run_qemu.sh`. Logic in a new CoRTOS module.
- why: C-model vs QEMU is a build target, not a second project file. Matches existing `cortos build` flags (`--ramstart`, `-O*`).
- alternatives:
  - Duplicate `config_qemu.yaml`: drifts.

## In-container qemu setup, binary in `build/`
- level: major-implementation-detail
- chosen: `setup_qemu.sh` inside `ajit_build_dev`. Output `$AJIT_HOME/build/qemu-ajit-v1.0/`. Host packages on `ajit_base` as needed. Not baked into the image.
- why: Same layout as toolchain `setup.sh` + Darwin build volume.
- alternatives:
  - Dockerfile `RUN` qemu: fights mount-then-build.

## Headless stdio + expected_uart.txt
- level: design-choice
- chosen: `-serial stdio -nographic`; grep all lines of `expected_uart.txt`; wall timeout; no gdb wait.
- why: Scriptable e2e. README pty/minicom/`-S` is interactive only.
- alternatives:
  - pty + minicom: not Goal evidence.

## qemu `-smp` matches CoRTOS thread count
- level: major-implementation-detail
- chosen: `run_qemu` passes `-smp Cores*ThreadsPerCore` (max 4) and `-m` from `TotalMemoryInKB` (floor 128MiB).
- why: qemu-ajit maps `cpu_index` to asr29 core/thread. Default `-smp 1` never runs AJIT thread (0,1).
- alternatives:
  - Hard-code `-smp 2`: wrong for `example_001`.

## qemu `ta 0` halts one CPU
- level: major-implementation-detail
- chosen: On AJIT1 (`CPU_FEATURE_ASR29`), trap 0x80 with traps disabled sets that CPU `halted` / `EXCP_HLT`. Do not `cpu_abort` the VM. Do not use LEON `TA0_SHUTDOWN` (that stops the whole guest).
- why: `cortos_exit` is `ta 0`. First thread to exit used to kill qemu before the second thread printed.
- alternatives:
  - Change CoRTOS `cortos_exit` on qemu to an infinite loop: would fork the C-model ABI.
