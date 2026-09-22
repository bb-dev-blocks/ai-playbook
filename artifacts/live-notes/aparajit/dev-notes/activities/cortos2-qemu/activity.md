# CoRTOS2 examples on qemu-ajit in the toolchain

| Key | Value |
|---|---|
| status | Active |
| slug | cortos2-qemu |
| branch | none |
| ticket | none |
| notes | |

# Goal

cortos2 examples in `repos/ajit-toolchain` build and run on **qemu-ajit** inside `ajit_build_dev`, with separate qemu scripts, while C-model `./build.sh` / `./run.sh` stay (or are fixed). All eleven examples pass **both** C-model and qemu. Coverage lives in `docs/cortos2-on-qemu-ajit.md`. Reuse existing qemu-ajit + `setup_qemu.sh` (no re-vendor).

# Scope

In: cortos2 modular `--target qemu` (link `0x00100000`, qemu UART, MMU off, `cortos_build_qemu/`); `build_qemu.sh` / `run_qemu.sh` / `expected_uart.txt` for `example_{001,005,050,100,150,155,200,210,250,310,320}` in that order; reuse in-tree qemu-ajit; in-tree qemu/cortos2/example fixes as needed so both platforms pass; C-model green for all eleven; cortos2 qemu doc.

Out: FPGA; NuttX; TFLite/ResNet; CoRTOS v1 edits; `validation_ladder` suites; baking qemu into Docker images; retargeting qemu-ajit to the FPGA memory/UART map; aparajit-root qemu lockstep; re-vendoring qemu.

# Background and Special Notes

- Sibling of Complete `cortos-qemu` (CoRTOS v1). Host: native arm64 `ajit_build_dev`, Ubuntu 24.04, Buildroot 2025 SPARC gcc.
- cortos2 CLI today: `cortos2 build` only (no `--target`). Examples: `./build.sh` → `cortos2 build`; `./run.sh` → `cortos_build/run_cmodel.sh`.
- qemu-ajit already vendored: RAM `0x00100000`, UART TX/RX `+0x04` / `+0x08`; asr29 + per-CPU `ta 0` from v1. Extend qemu only if a cortos2 example still fails.
- Hard cases likely: `210` ncram, `310` serial IRQ, `320` sw traps / VMAP (may need MMU revisit).
- C-model UART leftovers: `pkill -x ajit_C_system_m`.

# Current Design

- Product tree: `repos/ajit-toolchain` only. cortos2 under `os/rtos/cortos2`.
- QEMU: existing `$AJIT_HOME/qemu-ajit-v1.0` + `setup_qemu.sh` + `ajit_env` PATH. Not re-copied.
- Default `cortos2 build` = C-model (`cortos_build/`). `--target qemu` = new cortos2 module: link `0x00100000`, skip MMU enable, `cortos_build_qemu/`, headless run helper matching v1.
- `run_qemu.sh` → qemu `-M ajit1_generic -cpu AJIT1 -smp Cores*ThreadsPerCore -m` (floor 128MiB) `-nographic -serial stdio`; match every non-comment line of `expected_uart.txt`.
- Must not break: all eleven C-model `./build.sh && ./run.sh`.
- Doc: `docs/cortos2-on-qemu-ajit.md` (not the v1 doc).

# Current Plan

1. Add cortos2 `--target qemu` module + consts; prove on `example_001` (scripts + `expected_uart.txt`).
2. For each remaining example in order: C-model green, then qemu green; fix cortos2/qemu/example as needed; update ledger.
3. Final e2e: all eleven both paths; doc complete.

# Milestones

1. [x] cortos2 `--target qemu` module + `example_001` dual pass
   - tests:
     - default `cortos2 build` → `cortos_build/`
     - `--target qemu` → `cortos_build_qemu/` ELF linked at `0x00100000`
     - `001` C-model `./build.sh && ./run.sh` exit 0
     - `001` `./build_qemu.sh && ./run_qemu.sh` exit 0 (`expected_uart.txt`)
   - evidence:
     - `cd os/rtos/cortos2/examples/example_001 && ./build.sh && ./run.sh`
     - `./build_qemu.sh && ./run_qemu.sh`
     - `readelf -h cortos_build_qemu/main.elf` (or documented ELF name)

2. [x] `example_005` dual pass
   - tests: C-model + qemu exit 0
   - evidence: `cd .../example_005 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

3. [x] `example_050` dual pass
   - tests: C-model + qemu; qemu `-smp 2`; UART shows both threads as in `expected_uart.txt`
   - evidence: `cd .../example_050 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

4. [x] `example_100` dual pass
   - tests: C-model + qemu; lock / shared counter UART substrings
   - evidence: `cd .../example_100 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

5. [x] `example_150` dual pass
   - tests: C-model + qemu; queue UART substrings
   - evidence: `cd .../example_150 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

6. [x] `example_155` dual pass
   - tests: C-model + qemu; lockless queue UART substrings
   - evidence: `cd .../example_155 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

7. [x] `example_200` dual pass
   - tests: C-model + qemu; bget UART substrings
   - evidence: `cd .../example_200 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

8. [x] `example_210` dual pass
   - tests: C-model + qemu; ncram bget UART substrings
   - evidence: `cd .../example_210 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

9. [x] `example_250` dual pass
   - tests: C-model + qemu; queue + bget UART substrings
   - evidence: `cd .../example_250 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

10. [x] `example_310` dual pass
    - tests: C-model + qemu; serial IRQ path UART substrings
    - evidence: `cd .../example_310 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

11. [x] `example_320` dual pass
    - tests: C-model + qemu; sw traps / VMAP UART substrings
    - evidence: `cd .../example_320 && ./build.sh && ./run.sh`; `./build_qemu.sh && ./run_qemu.sh`

12. [x] Goal e2e
    - tests: all eleven C-model exit 0; all eleven qemu exit 0; doc has setup + link/UART + full ledger
    - evidence:
      - for each `example_*` above: `./build.sh && ./run.sh` and `./build_qemu.sh && ./run_qemu.sh`
      - `docs/cortos2-on-qemu-ajit.md`

# Next Steps

1. All eleven examples pass both paths. Ledger is in `docs/cortos2-on-qemu-ajit.md`.

# References

- `derived-from: cortos-qemu` — v1 qemu-ajit + CoRTOS `--target qemu` pattern; this sibling retargets to cortos2 and requires all eleven dual-pass.
- `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md` — context-only. Learned: setup, link `0x00100000`, UART offsets, headless `expected_uart.txt`, asr29 / `ta 0` notes.
- `repos/ajit-toolchain/os/rtos/cortos/src/cortos/sys/targets.py` — context-only. Learned: `BuildTarget` cmodel vs qemu (build dir, RAM, MMU, 16MB align).
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/driver.py` — context-only. Learned: `cortos2 build` has debug/opt flags only; no `--target` yet.
- `repos/ajit-toolchain/os/rtos/cortos2/examples/` — context-only. Learned: eleven examples with `build.sh`/`run.sh`/`config.yaml`; no qemu scripts yet.
