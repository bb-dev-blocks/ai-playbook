# Journal

## NuttX on qemu-ajit, one core and SMP (Active → Complete)

- Shipped: AJIT chip `arch/sparc/src/ajit1/` and board `boards/sparc/ajit1/ajit1-qemu/` (`nsh`, `smp`) in `repos/ajit-toolchain/nuttx` branch `ajit_main`, commit `04c56692be`. Toolchain commit `a7e584c07`: `nuttx` + `apps` submodules, `scripts/test-nuttx-ajit.sh`, `scripts/run-nuttx-ajit.sh`, QEMU UART fix. Aparajit doc commit `38844d9` (`docs/nuttx-on-ajit-qemu.md`); `docs/porting-nuttx.md` (layer-by-layer port walkthrough) written, untracked.
- Approach: clone `s698pm` (SMP LEON3); change only AJIT deltas. Link `0x00100000`; `-mcpu=v8`; CPU id from ASR29 (`core*2+thread`) instead of ASR17; removed S698PM extended-ack block (AJIT PIL 11 = IPI).
- SMP: QEMU starts all CPUs; secondaries park in `__start` on `.data` flags; `up_cpu_start` releases them; absent CPUs marked offline so one `NCPUS=4` image runs at `-smp 1..4`. IPI = toggle bit in INTC `IPI_INTR_MASK` with `IPI_INT_VAL` set.
- Discoveries: `libatomic` uses pthread mutexes → trap before syscalls; replaced with `ldstub` `__atomic_*_4`. Shared `sparc_v8_swint1.c` lacked `restore_critical_section` on context switch → SMP lock hang; fixed for all SPARC SMP. QEMU UART cleared `RX_FULL` per byte → burst input lost; fixed in `hw/char/ajit1_uart.c`. `bm3823` link break fixed to keep LEON regression green.
- Evidence: `./scripts/test-nuttx-ajit.sh` inside `ajit_build_dev` passed at close (LEON ×3, `nsh`, `smp -smp 2`, `smp -smp 4`).
- Accepted gaps: global atomic lock; shared offline TCB; device IRQs on CPU0 only; `marshal_updates` has no upstream; `repos/nuttx` still on disk.
