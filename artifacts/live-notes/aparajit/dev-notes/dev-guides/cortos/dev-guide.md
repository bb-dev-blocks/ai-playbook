# cortos — Dev-Guide

**CoRTOS** (Co-operative RTOS) for the **32-bit AJIT SPARC V8** processor. Programs are built from C sources plus a YAML project file; the `cortos` tool generates linker scripts, startup (`init.s`), trap handlers, and per-thread images. CoRTOS has been exercised extensively on AJIT hardware and the AJIT C system model.

This tree is a **copy** of `repos/ajit-toolchain/os/rtos/cortos/` at the aparajit repo root for direct use alongside `qemu-ajit-v1.0`, NuttX, and TFLM work.

## Notes

- **Scheduling model**: Co-operative — threads call into the scheduler (or return from `main`); no preemptive time-slicing in the classic sense. Stage 6 (current) supports N programs on X hardware threads with locks, queues, and optional `bget` heap (`AddBget: True` in YAML).
- **Build output**: `cortos build` creates `cortos_build/` with generated sources, linker scripts, and `build.sh`. User `.c` / `.h` files are copied flat into the build dir (no subdirectories).
- **Run target today**: `cortos_build/run_cmodel.sh` invokes **`ajit_C_system_model`** (AJIT cycle-accurate / functional C model), not `qemu-ajit`. QEMU integration is a follow-up; use `machine/ajit1/` for the qemu-ajit memory map when adapting run scripts.
- **Toolchain**: Generated build scripts expect a SPARC cross toolchain on `PATH` (historically `sparc-linux-gcc` / AJIT uClibc flow from `repos/ajit-toolchain` and `repos/gauri-tools`). In aparajit Docker, `sparc-gaisler-elf-gcc` (kernel.org nolibc alias) may work for compile-only experiments; full CoRTOS link/run still needs the AJIT-specific linker tooling and C model.
- **Python**: Entry point `./cortos` (Python 3.6+). Set `PYTHONPATH` to `cortos/src` when not installing as a package:

  ```bash
  export PYTHONPATH=/path/to/aparajit/cortos/src
  ./cortos build config.yaml
  ```

- **Memory / link bases**: Per-program linker template places `.text` at `0x0` and `.rodata`/`.data` at **`0x40000000`** (`res/programLinker.txt`) — the **FPGA / C-model** layout. This differs from **qemu-ajit** RAM at `0x00100000` (see `machine/ajit1/ajit1-machine.yaml`, profile `cortos_fpga` vs `nuttx_qemu`).
- **UART registers**: CoRTOS reference `res/references/Ajit_Device_Addresses.h` uses TX `0xFFFF3210`, RX `0xFFFF3220`. **qemu-ajit** models TX/RX at `+0x04` / `+0x08` from `0xFFFF3200`. Pick one layout per target (documented in `machine/ajit1/`).
- **Hardware limits** (enforced in `src/cortos/sys/config.py`): max **4 cores**, **2 threads per core**. Number of `ProgramThreads` entries must equal `Cores × ThreadsPerCore`.

## Quick start

```bash
cd cortos/examples/example_001
export PYTHONPATH=../../src
../../cortos build          # or ./build.sh
./run.sh                    # needs ajit_C_system_model on PATH
```

Example numbering (`examples/README.md`):

| Range | Scenario |
|-------|----------|
| 001–049 | Single core, single thread |
| 050–099 | Multi-thread, no collaboration |
| 100–149 | Multi-thread + locks |
| 150–199 | Multi-thread + queues |
| 200–249 | Multi-thread + bget malloc |
| 250–299 | Sync + queues + bget |

## Project layout

```
cortos/
  cortos                 # CLI entry (python3)
  README.md              # Design notes, memory layout, development stages
  dev-guide.md           # this file
  src/cortos/            # Python package: config, build, templates
    sys/                 # driver, build, config, compute
    files/               # Bottle templates (.tpl) → generated asm/C/linker
    common/              # elf utils, constants, bottle
  res/                   # Static asm snippets, linker samples, device headers
  examples/              # example_001 … example_250 (+ build.sh / run.sh each)
```

## `config.yaml` (minimal)

```yaml
Cores: 1
ThreadsPerCore: 1
TotalMemoryInKB: 4096
LogLevel: DEBUG   # optional: ALL | TRACE | DEBUG | INFO | ERROR | CRITICAL | NONE

ProgramThreads:
  - CortosInitCalls:
      - main
    StackSizeInBytes: 8192
```

Optional keys: `AddBget: True`, `TotalLockVars`, `LeastValidStackAddr`, per-thread `CortosLoopCalls` (infinite loop wrapper).

Build flags via CLI (`./cortos build -h`):

| Flag | Effect |
|------|--------|
| `-g` | Debug build; per-thread GDB ports from `-p` |
| `-O0` / `-O1` / `-O2` | Optimization level |
| `-s ADDR` | RAM start (must be 16 MiB aligned; default `0x0`) |

## Build flow

1. `cortos build [config.yaml]` reads YAML → `UserConfig`.
2. Creates `cortos_build/`, copies templates and user sources.
3. Computes memory map (locks, queues, heap, stacks, program regions) → generates `cortos.h` with addresses and API.
4. Runs `cortos_build/build.sh` (SPARC compile + link).

Inspect generated API: build any example, then read `cortos_build/cortos.h` (locks, queues, `cortos_printf`, `cortos_bget`, scratch `SHARED_INT_ADDR_*`, etc.). `examples/README.md` embeds a full sample.

## CoRTOS API (summary)

| Area | Functions / macros |
|------|-------------------|
| Locks | `cortos_lock_acquire`, `cortos_lock_release`, `cortos_reserveLockVar` |
| Queues | `cortos_writeMessage`, `cortos_readMessage`, `cortos_reserveQueue` |
| Heap | `cortos_bget`, `cortos_brel` (when `AddBget`) |
| Debug | `cortos_printf`, `cortos_exit`, `CORTOS_TRACE` / `CORTOS_DEBUG` / … |
| Time | `cortos_get_clock_time`, `cortos_sleep` |
| Shared ints | `SHARED_INT_ADDR_0` … (scratch region from generated header) |

Exit codes: `cortos_exit(0)` success; `1–255` reserved for CoRTOS; user errors `≥ 256`.

## CLI

```bash
export PYTHONPATH=cortos/src
./cortos build [options] [config.yaml]
./cortos show config [config.yaml]
```

Subcommands are implemented in `src/cortos/sys/driver.py`.

## Relationship to other aparajit trees

| Component | Role |
|-----------|------|
| `machine/ajit1/` | Canonical SoC map for NuttX / qemu-ajit / bare-metal |
| `qemu-ajit-v1.0/` | System QEMU; `-kernel` bare-metal ELF |
| `repos/nuttx/` | Stage-1 LEON boards; stage-2 AJIT board should consume `ajit1_machine.h` |
| `repos/ajit-toolchain/` | Upstream home of this RTOS; toolchain and C model |
| `repos/gauri-tools/` | Validation ladders, `compileToSparcUclibc.py`, linker scripts |
| `tflite-ajit.md` | TFLM bare-metal on qemu-ajit (separate from CoRTOS) |

## Follow-up integration (suggested)

1. Add `cortos_build/run_qemu.sh` using `machine/ajit1` RAM base and UART offsets, linking with a qemu-specific linker profile.
2. Wire `./aparajit-docker` helper to build a CoRTOS example and run under `qemu-system-sparc`.
3. Align or fork `res/programLinker.txt` for `nuttx_qemu` profile when the same sources must run on QEMU and C model.

## Artifacts

| Name | Description |
|------|-------------|
| `cortos` | CLI entry script |
| `src/cortos/` | Python build system and templates |
| `res/` | `init.s`, trap handlers, `Ajit_Device_Addresses.h`, linker samples |
| `examples/` | Runnable projects (`config.yaml` + `main.c` + `build.sh` / `run.sh`) |
| `README.md` | Internal design notes and stage history |
| `machine/ajit1/ajit1-machine.yaml` | Shared AJIT machine definition (NuttX / QEMU) |

## Known pitfalls

- **`ajit_C_system_model` missing** — `run.sh` fails unless AJIT toolchain environment is installed (`repos/ajit-toolchain`).
- **Flat source tree** — only `*.c` in the project root are picked up; no subdirs.
- **Program count** — must match hardware thread count exactly.
- **Link base mismatch** — do not load a `0x40000000`-linked CoRTOS ELF with qemu-ajit `-kernel` without relinking for `0x00100000`.
- **Original path** — changes may need syncing back to `repos/ajit-toolchain/os/rtos/cortos/` if that submodule is the long-term upstream.
