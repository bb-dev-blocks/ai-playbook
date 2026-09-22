# CoRTOS examples on qemu-ajit in the toolchain

| Key | Value |
|---|---|
| status | Complete |
| slug | cortos-qemu |
| branch | none |
| ticket | none |
| notes | |

# Goal

CoRTOS examples in `repos/ajit-toolchain` build and run on **qemu-ajit** inside `ajit_build_dev`, with separate qemu scripts, while C-model `./build.sh` / `./run.sh` stay. QEMU source is vendored in the toolchain and built in-container. Coverage (and skip reasons) live in `docs/cortos-on-qemu-ajit.md`.

# Scope

In: copy `qemu-ajit-v1.0` **source** into the toolchain; `setup_qemu.sh` + `ajit_env` PATH; modular CoRTOS `--target qemu`; `build_qemu.sh` / `run_qemu.sh` / `expected_uart.txt` / `cortos_build_qemu/` for eight examples; incremental qemu e2e (`001` first, then `stads` → `050` → `100` → `150` → `200` → `250` → `sort`); skip doc; C-model five still green.

Out: FPGA, NuttX, TFLite, aparajit-root qemu / `aparajit-docker` / root `cortos/` copy, baking qemu into Docker images, changing C-model script behavior, retargeting qemu-ajit to the FPGA memory/UART map.

# Background and Special Notes

- Sibling of completed `ajit-toolchain-setup` (C-model only). Host: native arm64 `ajit_build_dev`, Ubuntu 24.04, Buildroot 2025 SPARC gcc.
- qemu-ajit README: single-core functional; `-kernel` wants baremetal ELF; documented run uses pty/`-S` (not Goal).
- CoRTOS C-model: `.text` at `0x0`, data `0x40000000`, UART TX/RX `0xFFFF3210` / `0xFFFF3220`. qemu-ajit: RAM `0x00100000`, UART TX/RX `+0x04` / `+0x08`.
- C-model `--ramstart` is 16MB-aligned. QEMU link `0x00100000` is not. **Later:** maybe move qemu RAM to 16MB-aligned and share one rule.
- `050+` need qemu `-smp 2` (AJIT `cpu_index` → asr29 core/thread). `cortos_exit` is `ta 0`; AJIT1 must halt **that** CPU, not `cpu_abort` the VM.
- qemu `helper_rdasr29` returns `0x50520ctt` (CPU0 = `0x50520000`). Old qemu returned 0 → trap 0x80 before `main`.
- Copy size ~165M source; do not vendor `build/` (~825M).
- C-model UART leftovers: `pkill -x ajit_C_system_m`. Do not `pkill -f ajit_C_system_model` from a line containing that string.

# Current Design

- Product tree: `repos/ajit-toolchain` only. Aparajit-root `qemu-ajit-v1.0/` is a one-shot donor, not kept in lockstep.
- QEMU source `$AJIT_HOME/qemu-ajit-v1.0`; binary `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` via `setup_qemu.sh` + `docker/ajit_build/ajit_env`. Host packages on `docker/ajit_base/setup_ajit_base.sh`.
- Default `cortos build` is C-model (`cortos_build/`, RAM `0x0`, MMU on). `--target qemu` is `os/rtos/cortos/src/cortos/sys/targets.py` + `qemu.py`: link `0x00100000`, skip MMU enable, `genVmapAsm` still at `0x0`, ELF `e_entry` → `_start`, `cortos_build_qemu/`.
- `run_qemu.sh` → `cortos run --target qemu`: `-M ajit1_generic -cpu AJIT1 -smp Cores*ThreadsPerCore -m` from `TotalMemoryInKB` (floor 128MiB) `-nographic -serial stdio`; match every non-comment line of `expected_uart.txt`.
- Must not break: C-model `example_{001,050,100,150,250}` `./build.sh && ./run.sh` inside `ajit_build_dev`.
- Touched (shipped): `qemu-ajit-v1.0/` (incl. `helper_rdasr29`, `ta 0` per-CPU halt in `target/sparc/int32_helper.c`), `setup_qemu.sh`, CoRTOS qemu module + init/vmap/build templates, eight example `build_qemu.sh`/`run_qemu.sh`/`expected_uart.txt`, `docs/cortos-on-qemu-ajit.md`, small UART prints in `example_050`/`example_sort`, `#if 0` on leftover `example_sort/qsort.c`. Pins: toolchain `1fcd563fd`; parent `77ce38c`.

# Current Plan

Shipped. Reopen only via `resume-work`.

# Milestones

1. [x] qemu-ajit source + in-container build
   - tests: source at `qemu-ajit-v1.0/` without committed `build/`; `setup_qemu.sh` → `qemu-system-sparc`; `-M ajit1_generic`
   - evidence: inside `ajit_build_dev`, `source ./set_ajit_home`; `source docker/ajit_build/ajit_env`; `./setup_qemu.sh`; `qemu-system-sparc -M help | grep ajit1_generic`

2. [x] CoRTOS `--target qemu` module + example scripts
   - tests: default `cortos build` → `cortos_build/`; `--target qemu` → `cortos_build_qemu/main.elf` at `0x00100000`; eight `build_qemu.sh`/`run_qemu.sh`; C-model `001` still green
   - evidence: `cd os/rtos/cortos/examples/example_001 && ./build.sh && ./run.sh`; `./build_qemu.sh`; `readelf -h cortos_build_qemu/main.elf`

3. [x] `example_001` on qemu (exclusive)
   - tests: `expected_uart.txt` has `Hello There`; `./build_qemu.sh && ./run_qemu.sh` exit 0
   - evidence: `cd os/rtos/cortos/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`

4. [x] `example_stads` on qemu
   - tests: UART `Number of Stars:`
   - evidence: `cd os/rtos/cortos/examples/example_stads && ./build_qemu.sh && ./run_qemu.sh`

5. [x] `example_050` on qemu
   - tests: `-smp 2`; UART `050 thread 0,0` and `050 thread 0,1`
   - evidence: `cd os/rtos/cortos/examples/example_050 && ./build_qemu.sh && ./run_qemu.sh`

6. [x] `example_100` on qemu
   - tests: UART `VALUE_IS: 20`
   - evidence: `cd os/rtos/cortos/examples/example_100 && ./build_qemu.sh && ./run_qemu.sh`

7. [x] `example_150` on qemu
   - tests: UART `Sending Message 1` and `Received Message`
   - evidence: `cd os/rtos/cortos/examples/example_150 && ./build_qemu.sh && ./run_qemu.sh`

8. [x] `example_200` on qemu
   - tests: UART both thread-finished lines
   - evidence: `cd os/rtos/cortos/examples/example_200 && ./build_qemu.sh && ./run_qemu.sh`

9. [x] `example_250` on qemu
   - tests: UART `Acquiring Memory!` and `Received Message!`; do not `ldstub`-spin C-model reader if re-run
   - evidence: `cd os/rtos/cortos/examples/example_250 && ./build_qemu.sh && ./run_qemu.sh`

10. [x] `example_sort` on qemu
    - tests: longer timeout allowed; skip if blocked
    - evidence: skip row in `docs/cortos-on-qemu-ajit.md` (`./run_qemu.sh` 300s, no UART)

11. [x] Goal e2e
    - tests: every in-scope example qemu pass or skip row; C-model five still exit 0; doc has setup + link/UART + later 16MB note
    - evidence: seven `./run_qemu.sh` exit 0 + sort skip; `example_{001,050,100,150,250}` `./build.sh && ./run.sh`; `docs/cortos-on-qemu-ajit.md`

# Next Steps

1. Push toolchain `marshal_updates` (`1fcd563fd`) and aparajit (`77ce38c`) when asked.
2. Fastest check: inside `ajit_build_dev`, `source` `ajit_env`; `cd os/rtos/cortos/examples/example_001 && ./run_qemu.sh` (after `./build_qemu.sh` once).
3. Safest first edit: do not commit `cortos_build_qemu/` or `$AJIT_HOME/build/qemu-ajit-v1.0/`; do not retarget qemu RAM/UART to the FPGA map without a sibling.
4. Optional later: `example_sort` longer timeout or smaller SIZE; 16MB-aligned qemu RAM; aparajit-root qemu lockstep (out of scope).

# References

- `derived-from: ajit-toolchain-setup` — C-model Docker/Buildroot/CoRTOS baseline; qemu was out of that scope.
- `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md` — setup + coverage ledger.
- Pins: `repos/ajit-toolchain` `1fcd563fd`; aparajit `77ce38c`.
- `qemu-ajit-v1.0/README.md` — context-only (`-M ajit1_generic`, `-kernel`, pty/`-S`).
- `repos/ajit-toolchain/os/rtos/cortos/src/cortos/sys/qemu.py` — headless run helper.
