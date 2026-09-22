# Requirement Definition

## Example scripts: C-model or QEMU
- kind: end-user-interface
- Each in-scope example keeps `./build.sh` / `./run.sh` as C-model (no behavior change).
- Each adds `./build_qemu.sh` / `./run_qemu.sh`.
- QEMU output dir is `cortos_build_qemu/` (does not clobber `cortos_build/`).

## `cortos build --target qemu`
- kind: software-interface
- Default `cortos build` stays C-model (RAM start `0x0`, FPGA/C-model UART).
- `--target qemu` links at `0x00100000`, uses qemu-ajit UART (`TX +0x04`, `RX +0x08` from `0xFFFF3200`), writes `cortos_build_qemu/`.
- `build_qemu.sh` is a thin wrapper around that flag.

## Headless QEMU run
- kind: end-user-interface
- `run_qemu.sh` runs `qemu-system-sparc -M ajit1_generic -cpu AJIT1 -nographic -serial stdio`.
- No `-S`, no gdb unless an explicit debug flag.
- `-kernel` is `cortos_build_qemu/main.elf`.
- Exit 0 iff every substring in that example’s `expected_uart.txt` appears on stdout before timeout.
- Default timeout ~30s; longer allowed via per-example run notes (`sort` likely). Timeout or missing text → fail.

## QEMU built in toolchain container
- kind: end-user-interface
- Source lives in `repos/ajit-toolchain/qemu-ajit-v1.0/` (copy of aparajit-root tree, **no** `build/`).
- Build **inside** `ajit_build_dev` via `setup_qemu.sh` (or a named optional step from `setup.sh`), not in the Dockerfile.
- Artifacts under `$AJIT_HOME/build/qemu-ajit-v1.0/`. `ajit_env` puts `qemu-system-sparc` on `PATH`.
- Document the flow in `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md` and setup notes.

## Incremental QEMU coverage
- kind: end-user-interface
- First QEMU milestone is **only** `example_001`. Then add, in order: `stads`, `050`, `100`, `150`, `200`, `250`, `sort`.
- If an example cannot run, record it in `docs/cortos-on-qemu-ajit.md` with the reason. No silent skip.
- In-scope dirs: those eight only.

## C-model still works
- kind: end-user-interface
- After CoRTOS changes, `example_{001,050,100,150,250}` still `./build.sh && ./run.sh` exit 0 on the C-model.

## Modular CoRTOS qemu path
- kind: internal-behavior
- QEMU device map, link rules, output dir, and run helper live in a new CoRTOS module, same layering as the rest of CoRTOS. No scattered `if qemu` in the C-model path.
