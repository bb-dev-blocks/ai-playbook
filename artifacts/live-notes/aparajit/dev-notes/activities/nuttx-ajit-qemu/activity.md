# NuttX on ajit-qemu

| Key | Value |
|---|---|
| status | Complete |
| slug | nuttx-ajit-qemu |
| branch | none |
| ticket | none |
| notes | |

# Goal

NuttX builds and runs on qemu-ajit from submodules inside `repos/ajit-toolchain`, inside `ajit_build_dev`. `ajit1-qemu:nsh` at `-smp 1` answers `help` and `hello` on the UART. `ajit1-qemu:smp` shows every CPU online at `-smp 2` and `-smp 4`. A terminal can type simple NSH commands. The process is `docs/nuttx-on-ajit-qemu.md`. `s698pm-dkit:nsh`, `s698pm-dkit:smp`, and `xx3823:nsh` still cross-compile.

# Scope

In: `repos/ajit-toolchain/nuttx` tracking `git@github.com:bb-dev-blocks/nuttx.git`, plus sibling `apps`; AJIT chip and `ajit1-qemu` board (link `0x00100000`, qemu UART); build and run in `ajit_build_dev` with that tree's SPARC gcc and qemu-ajit; scripted UART checks; interactive NSH; LEON cross-compile regression; process doc.

Out: deleting `repos/nuttx`; rewriting upstream NuttX authors; `aparajit-docker` as the build path; booting LEON images on qemu-ajit; FPGA; TFLite; more than 4 CPUs.

# Background and Special Notes

- Root dev-guide treats NuttX-on-AJIT as stage-2. `cortos2-qemu` proved the machine map and left NuttX out.
- Images `ajit_base:1.0` / `ajit_build_dev:1.0` come from Complete `ajit-toolchain-setup` (Ubuntu 24.04, Buildroot 2025.02.18, `sparc-linux-gcc` 13.4.0). Do not rebuild. Container bind-mounts the toolchain repo at `/home/ajit/ajit-toolchain`.
- `repos/nuttx` (old super-repo with Anshuman commits) is unused. Its commits were not cherry-picked. Removal is later work.
- QEMU `hw/sparc/ajit1.c` matches between repo-root `qemu-ajit-v1.0/` and `repos/ajit-toolchain/qemu-ajit-v1.0/`. Run the toolchain build's `qemu-system-sparc`.

# Current Design

Shipped. Layer-by-layer walkthrough with snippets: `docs/porting-nuttx.md`.

- Submodules in `repos/ajit-toolchain`: `nuttx` → `git@github-bb:bb-dev-blocks/nuttx.git`, branch `ajit_main` (fork `master` `a31f7b7988` + AJIT commits; port is `04c56692be`). `apps` → Apache nuttx-apps at `bc0ed23a5dea42e9aa42dfccc372fe9f4654d10d`. Toolchain commit `a7e584c07` adds both plus scripts. Doc commit in aparajit: `38844d9`.
- Chip `arch/sparc/src/ajit1/`, cloned from `s698pm`. Board `boards/sparc/ajit1/ajit1-qemu/` with `nsh` and `smp` configs. Link and `CONFIG_RAM_START` `0x00100000`, 16 MiB.
- Shared-code edits: `sparc_v8/Toolchain.defs` (`-mcpu=v8` for AJIT1); `sparc_v8/sparc_v8_swint1.c` calls `restore_critical_section` on restore/switch context (SMP lock fix, affects LEON SMP); `bm3823` link fix.
- QEMU edit: `qemu-ajit-v1.0/hw/char/ajit1_uart.c` keeps `RX_FULL` while FIFO non-empty.
- Invariants (must not break):
  - CPU index = ASR29 `core*2 + thread` in `ajit1_head.S`, `ajit1_exceptions.S`, `ajit1_cpuindex.c` — all three agree.
  - IRQ number = SPARC trap number; external PIL n = trap `0x10+n` (timer `0x1a`, IPI `0x1b`, UART `0x1c`).
  - INTC control register is per accessing CPU. CPU0 owns timer/UART levels; secondaries enable IPI only (`ajit1_cpu_boot`).
  - `g_cpu_present` / `g_cpu_release` stay in `.data` (CPU0 zeroes BSS while secondaries spin).
  - QEMU starts all CPUs; secondaries park in `__start`. Absent CPUs → `ajit1_mark_offline` (one image, `CONFIG_SMP_NCPUS=4`, runs at `-smp 1..4`).
  - `ajit1-atomic.c` provides `__atomic_*_4` via `ldstub`; do not link `libatomic` (pthread mutex traps before syscalls exist).
  - Never `-mcpu=leon3` for AJIT.
- Run: `./scripts/run-nuttx-ajit.sh nsh|smp` (quit Ctrl-A X). Scripted checks use TCP serial.

# Current Plan

Done. No open plan.

# Milestones

1. [x] Toolchain submodules track the fork, with no Anshuman in `nuttx` history
   - evidence:
     - `git -C repos/ajit-toolchain submodule status nuttx apps`
     - `git -C repos/ajit-toolchain/nuttx remote get-url origin` → `git@github-bb:bb-dev-blocks/nuttx.git`
     - `git -C repos/ajit-toolchain/nuttx log --format='%an <%ae>%n%cn <%ce>' | rg -i anshuman` (no output)

2. [x] Three LEON configs cross-compile in `ajit_build_dev`
   - evidence: inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh leon` (`OK s698pm-dkit:nsh`, `OK s698pm-dkit:smp`, `OK xx3823:nsh`)

3. [x] `ajit1-qemu:nsh` at `-smp 1` answers `help` and `hello`; manual terminal command exists
   - evidence: `./scripts/test-nuttx-ajit.sh nsh` (`OK ajit1-qemu:nsh`); manual `./scripts/run-nuttx-ajit.sh nsh`

4. [x] `ajit1-qemu:smp` brings up every CPU at `-smp 2` and `-smp 4`
   - evidence: `./scripts/test-nuttx-ajit.sh smp` (`OK ajit1-qemu:smp -smp 2` with `ps`; `OK ajit1-qemu:smp -smp 4`); manual `./scripts/run-nuttx-ajit.sh smp`

5. [x] End-to-end: LEON builds, single-core NSH, both SMP widths, and the process doc
   - evidence: inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh` — passed at close (all five OK lines); `test -f docs/nuttx-on-ajit-qemu.md`; `test -f docs/porting-nuttx.md`

# Next Steps

Minor-fix runway:

1. Fastest check after any edit: inside `ajit_build_dev`, `./scripts/test-nuttx-ajit.sh nsh` (~1 min), then `smp`, then full run (~4 min).
2. `docs/porting-nuttx.md` is untracked in aparajit; commit when the user asks.
3. `repos/ajit-toolchain` branch `marshal_updates` has no upstream; `a7e584c07` may be unpushed.
4. Known shortcuts to revisit if they matter: single global atomic lock (`ajit1-atomic.c`); one shared offline TCB (`ajit1_cpustart.c`); device IRQs only on CPU0.
5. `docs/nuttx-on-ajit-qemu.md` table still says fork branch `master`; submodule actually follows `ajit_main`.

# References

- `machine/ajit1/ajit1-machine.yaml` — context-only. Learned: RAM link `0x00100000`, UART0 `0xFFFF3200`, machine `ajit1_generic`, CPU `AJIT1`.
- `machine/ajit1/include/ajit1_machine.h` — context-only. Learned: C constants for RAM, UART register offsets, and the qemu machine name.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/qemu.py` — context-only. Learned: run argv `-M ajit1_generic -cpu AJIT1 -smp N -m 128M -serial stdio -nographic -kernel`, SMP cap 4.
- `repos/ajit-toolchain/docker/ajit_build_dev/run.sh` — context-only. Learned: bind-mount of the whole toolchain repo at `/home/ajit/ajit-toolchain`.
- `repos/ajit-toolchain/.gitmodules` — context-only. Learned: existing submodule pattern is top-level paths with `git@github.com:bb-dev-blocks/...`.
- `nuttx-ajit.md` — context-only. Learned: stage-1 LEON configs and the old aparajit-docker path. This activity does not use that Docker path.
- `.dev-notes/activities/ajit-toolchain-setup/activity.md` — context-only. Learned: reuse `ajit_base:1.0` and `ajit_build_dev:1.0`; do not rebuild them.
- `.dev-notes/activities/cortos2-qemu/activity.md` — context-only. Learned: qemu parameters and that NuttX was explicitly out of that activity.
