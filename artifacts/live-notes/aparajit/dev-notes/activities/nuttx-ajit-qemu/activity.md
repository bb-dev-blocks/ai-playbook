# NuttX on ajit-qemu

| Key | Value |
|---|---|
| status | Active |
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

- Root dev-guide already treats NuttX-on-AJIT as stage-2. `cortos2-qemu` left NuttX out and proved the machine map.
- Use the existing `ajit_base:1.0` / `ajit_build_dev:1.0` images from Complete `ajit-toolchain-setup` (Ubuntu 24.04, Buildroot 2025.02.18, `sparc-linux-gcc` 13.4.0, distro `python3`). Do not rebuild those images. Same container already builds cortos2, TFLite, and AHIR. It bind-mounts the toolchain repo at `/home/ajit/ajit-toolchain`.
- `bb-dev-blocks/nuttx` `master` is `a31f7b7988` (63436 commits, original authors, no Anshuman). No `bb-dev-blocks` nuttx-apps repo.
- `repos/nuttx` `main` is six Anshuman commits. Kernel checkout has two more, not on the fork. Live tree is a fresh clone. The bm3823 link fix (drop undefined `PRE_STACK_FRAME_*` uses; include `nuttx/spinlock.h`) is applied uncommitted on the toolchain `nuttx` tree. Do not cherry-pick the old Anshuman commits.
- QEMU source `hw/sparc/ajit1.c` matches between repo-root `qemu-ajit-v1.0/` and `repos/ajit-toolchain/qemu-ajit-v1.0/`. Run the toolchain build's `qemu-system-sparc`.

# Current Design

- Submodules: `repos/ajit-toolchain/nuttx` → `git@github-bb:bb-dev-blocks/nuttx.git` `master`. `repos/ajit-toolchain/apps` → `https://github.com/apache/nuttx-apps.git` at `bc0ed23a5dea42e9aa42dfccc372fe9f4654d10d` (sibling `../apps` from the kernel tree).
- `repos/nuttx` stays on disk, unused by this activity. Removal is later.
- New commits: author and committer `bb-dev-blocks <basicblocksdevelopers@gmail.com>`. Upstream authors stay. Live `nuttx` history has no Anshuman author or committer.
- Chip `arch/sparc/src/ajit1/`, board `boards/sparc/ajit1/ajit1-qemu/` with `nsh` and `smp`, patterned on `s698pm-dkit`. Link origin `0x00100000`. UART0 `0xFFFF3200` (`ctrl +0x00`, `tx +0x04`, `rx +0x08`).
- Build: `CROSSDEV=sparc-linux-` inside the existing `ajit_build_dev` container. Do not rebuild the image. Manual run: `./scripts/run-nuttx-ajit.sh nsh` (`-smp 1`) and `./scripts/run-nuttx-ajit.sh smp` (`-smp 2`). Quit: Ctrl-A then X. Scripted checks use TCP serial.
- Doc: `docs/nuttx-on-ajit-qemu.md`. `nuttx-ajit.md` stays the old handoff.

# Current Plan

1. Add both submodules. Confirm `nuttx` `origin` is the bb-dev-blocks fork and `git log` has no Anshuman.
2. Cross-compile `s698pm-dkit:nsh`, `s698pm-dkit:smp`, `xx3823:nsh` in `ajit_build_dev`. Fix the bm3823 build only if that config fails.
3. Add `ajit1-qemu:nsh`. Scripted `-smp 1` must show `help` and `hello`. Manual command attaches a terminal.
4. Add `ajit1-qemu:smp`. Scripted `-smp 2` and `-smp 4` must show every extra CPU online. Manual `-smp 2` accepts `ps`.
5. One e2e script runs the LEON builds plus the three qemu checks. Doc covers checkout, identity, build, scripted runs, and the manual terminal.

# Milestones

1. [x] Toolchain submodules track the fork, with no Anshuman in `nuttx` history
   - tests:
     - `nuttx` origin URL is `git@github.com:bb-dev-blocks/nuttx.git` (or `git@github-bb:bb-dev-blocks/nuttx.git`)
     - `apps` submodule present at the pinned sha
     - author and committer lines contain no `Anshuman`
   - evidence:
     - `git -C repos/ajit-toolchain submodule status`
     - `git -C repos/ajit-toolchain/nuttx remote get-url origin`
     - `git -C repos/ajit-toolchain/nuttx log --format='%an <%ae>%n%cn <%ce>' | rg -i anshuman` (no output)

2. [x] Three LEON configs cross-compile in `ajit_build_dev`
   - tests:
     - `s698pm-dkit:nsh`, `s698pm-dkit:smp`, `xx3823:nsh` each produce `ELF 32-bit MSB SPARC`
   - evidence:
     - inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh leon`

3. [x] `ajit1-qemu:nsh` at `-smp 1` answers `help` and `hello`; manual terminal command exists
   - tests:
     - scripted UART contains `nsh>` command output for `help` and `hello`
     - `./scripts/run-nuttx-ajit.sh nsh` is the interactive terminal (no timeout harness)
   - evidence:
     - inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh nsh`
     - `./scripts/run-nuttx-ajit.sh nsh` (manual: `help`, `hello`, Ctrl-A X)

4. [x] `ajit1-qemu:smp` brings up every CPU at `-smp 2` and `-smp 4`
   - tests:
     - `-smp 2`: CPU1 online on UART
     - `-smp 4`: CPU1, CPU2, CPU3 online on UART
     - manual `-smp 2`: `ps` at `nsh>`
   - evidence:
     - inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh smp`
     - `./scripts/run-nuttx-ajit.sh smp` (manual, `-smp 2`)

5. [x] End-to-end: LEON builds, single-core NSH, both SMP widths, and the process doc
   - tests:
     - e2e: milestones 2–4 scripted checks pass in one run
     - doc exists and names the manual `help` / `hello` / `ps` session
   - evidence:
     - inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh`
     - `test -f docs/nuttx-on-ajit-qemu.md`

# Next Steps

1. Commit when this activity is closed. Nothing else is open.

# References

- `machine/ajit1/ajit1-machine.yaml` — context-only. Learned: RAM link `0x00100000`, UART0 `0xFFFF3200`, machine `ajit1_generic`, CPU `AJIT1`, NuttX port not started.
- `machine/ajit1/include/ajit1_machine.h` — context-only. Learned: C constants for RAM, UART register offsets, and the qemu machine name.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/qemu.py` — context-only. Learned: run argv `-M ajit1_generic -cpu AJIT1 -smp N -m 128M -serial stdio -nographic -kernel`, SMP cap 4.
- `repos/ajit-toolchain/docker/ajit_build_dev/run.sh` — context-only. Learned: bind-mount of the whole toolchain repo at `/home/ajit/ajit-toolchain`.
- `repos/ajit-toolchain/.gitmodules` — context-only. Learned: existing submodule pattern is `ahir` and `tflite-micro` as top-level paths with `git@github.com:bb-dev-blocks/...`.
- `nuttx-ajit.md` — context-only. Learned: stage-1 LEON configs and the old aparajit-docker path. This activity does not use that Docker path.
- `.dev-notes/activities/ajit-toolchain-setup/activity.md` — context-only. Learned: shipped image is Ubuntu 24.04 + Buildroot 2025.02.18 (`sparc-linux-gcc` 13.4.0, distro `python3` 3.12.3). Reuse `ajit_base:1.0` and `ajit_build_dev:1.0`; do not rebuild them. Later cortos2, TFLite, and AHIR builds use this same image.
- `.dev-notes/activities/cortos2-qemu/activity.md` — context-only. Learned: qemu parameters and that NuttX was explicitly out of that activity.
