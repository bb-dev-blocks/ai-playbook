# Design Decisions

## Sibling of cortos-qemu; product is cortos2
- level: design-choice
- chosen: New activity `cortos2-qemu`. Target `os/rtos/cortos2` only. Reuse vendored qemu-ajit + `setup_qemu.sh` from completed `cortos-qemu`. Do not edit CoRTOS v1 for this work.
- why: Same qemu story; different RTOS tree and CLI (`cortos2`).
- alternatives:
  - Reopen `cortos-qemu`: wrong product; parent is Complete.
  - Re-vendor qemu: duplicate ~165M; already in toolchain.

## Two builds, same sources
- level: design-choice
- chosen: Separate qemu image (`cortos_build_qemu/`) with qemu RAM/UART. C-model `./build.sh` / `./run.sh` stay unless broken and fixed.
- why: Layout mismatch (RAM `0x0` + FPGA UART vs RAM `0x00100000` + qemu UART).
- alternatives:
  - Teach qemu the FPGA/C-model map: bigger machine change; deferred / out.

## `cortos2 build --target qemu`
- level: design-choice
- chosen: CLI target flag; one `config.yaml` per example. Thin `build_qemu.sh` / `run_qemu.sh`. Logic in a new cortos2 module (mirror v1 `targets.py` / `qemu.py`).
- why: C-model vs QEMU is a build target. Matches v1 UX; cortos2 has no `--target` today.
- alternatives:
  - Duplicate `config_qemu.yaml`: drifts.
  - Separate `cortos2-qemu` binary: extra PATH surface.

## QEMU link at 0x00100000
- level: major-implementation-detail
- chosen: qemu target links at `0x00100000` to match qemu-ajit RAM. Do not apply C-model 16MB-alignment to this target.
- why: Matches `ajit1_generic` today (same as v1).
- alternatives:
  - Move qemu RAM to `0x01000000` and share one rule: later; note in docs.

## QEMU skips CoRTOS MMU enable
- level: major-implementation-detail
- chosen: qemu target leaves MMU off after boot (physical map at `0x00100000`). C-model still enables MMU as today.
- why: qemu-ajit PROM occupies `0`..`1MiB`; enabling a `0x0` page-table stub after a `0x00100000` link would unmap the image.
- alternatives:
  - Full SRMMU tables for qemu RAM: later if an example needs MMU (`320` may force revisit).

## Headless stdio + expected_uart.txt
- level: design-choice
- chosen: `-serial stdio -nographic`; grep all lines of `expected_uart.txt`; wall timeout; no gdb wait.
- why: Scriptable e2e. Same as v1.
- alternatives:
  - Reuse `main.results` C-model format: not UART stdout.

## qemu `-smp` matches thread count
- level: major-implementation-detail
- chosen: run helper passes `-smp Cores*ThreadsPerCore` (cap as in v1) and `-m` from configured memory (floor 128MiB).
- why: qemu-ajit maps `cpu_index` to asr29 core/thread.
- alternatives:
  - Hard-code `-smp 2`: wrong for single-thread examples.

## Reuse v1 qemu asr29 / `ta 0` behavior
- level: major-implementation-detail
- chosen: Keep existing qemu-ajit `helper_rdasr29` and per-CPU `ta 0` halt. Change further only if a cortos2 example still fails.
- why: Already required for multi-thread CoRTOS on qemu; avoiding re-litigation.
- alternatives:
  - Mask asr29 / change `cortos_exit` in cortos2 only: forks C-model ABI.

## All eleven examples; both targets required
- level: design-choice
- chosen: Order `001 → 005 → 050 → 100 → 150 → 155 → 200 → 210 → 250 → 310 → 320`. Each must pass C-model and qemu. Fix in-tree qemu/cortos2/examples as needed. No skip-as-done.
- why: User: keep all tests; one-by-one; both platforms green.
- alternatives:
  - Subset ladder + skip doc: rejected.
  - validation_ladder suites: out of scope.

## Separate cortos2 qemu doc
- level: design-choice
- chosen: `docs/cortos2-on-qemu-ajit.md` for setup pointer + coverage ledger. Leave `docs/cortos-on-qemu-ajit.md` for v1. Optional cross-link.
- why: Two products; one ledger each.
- alternatives:
  - Merge into v1 doc: mixes trees and example sets.

## In-scope qemu/cortos2 fixes for hard examples
- level: design-choice
- chosen: IRQ (`310`), traps/VMAP (`320`), ncram (`210`), etc. may need qemu-ajit or cortos2 changes inside the toolchain. Do them here.
- why: Done means both platforms pass; cannot waive.
- alternatives:
  - Cap at v1 qemu only; sibling for hard cases: rejected.

## cortos2 runs on distro Python
- level: major-implementation-detail
- chosen: `cortos2` shebang is `python3`. Bottle uses `getfullargspec` when `getargspec` is gone. ELF parsing uses the image `python3-pyelftools`, not vendored `pyelftools-0.25` and not the Python 3.6 prefix.
- why: `ajit_build_dev` is Ubuntu 24.04; distro Python is 3.12. `inspect.getargspec` and `pyelftools-0.25` do not run there.
- alternatives:
  - Keep `python3.6` plus vendored elftools: second interpreter for one tool.
