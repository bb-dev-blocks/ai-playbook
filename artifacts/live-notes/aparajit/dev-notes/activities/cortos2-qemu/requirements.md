# Requirement Definition

## Example scripts: C-model or QEMU
- kind: end-user-interface
- Each in-scope example keeps `./build.sh` / `./run.sh` as C-model (no behavior change unless broken and fixed).
- Each adds `./build_qemu.sh` / `./run_qemu.sh`.
- QEMU output dir is `cortos_build_qemu/` (does not clobber `cortos_build/`).

## `cortos2 build --target qemu`
- kind: software-interface
- Default `cortos2 build` stays C-model (RAM start `0x0`, FPGA/C-model UART, `cortos_build/`).
- `--target qemu` links at `0x00100000`, uses qemu-ajit UART (`TX +0x04`, `RX +0x08` from `0xFFFF3200`), writes `cortos_build_qemu/`.
- `build_qemu.sh` is a thin wrapper around that flag.

## Headless QEMU run
- kind: end-user-interface
- `run_qemu.sh` runs `qemu-system-sparc -M ajit1_generic -cpu AJIT1 -nographic -serial stdio`.
- No `-S`, no gdb unless an explicit debug flag.
- `-kernel` is `cortos_build_qemu/main.elf` (or the cortos2-equivalent ELF name if different; document actual path).
- Exit 0 iff every substring in that example’s `expected_uart.txt` appears on stdout before timeout.
- Default timeout ~30s; longer allowed via per-example run notes. Timeout or missing text → fail.

## Reuse toolchain qemu-ajit
- kind: end-user-interface
- Do not re-vendor qemu. Use existing `repos/ajit-toolchain/qemu-ajit-v1.0/` + `setup_qemu.sh` + `ajit_env` PATH.
- Document cortos2-specific flow in `repos/ajit-toolchain/docs/cortos2-on-qemu-ajit.md` (setup may point at `docs/cortos-on-qemu-ajit.md`).

## Incremental coverage — all eleven examples
- kind: end-user-interface
- In-scope dirs (order): `example_001`, `005`, `050`, `100`, `150`, `155`, `200`, `210`, `250`, `310`, `320`.
- First QEMU milestone is **only** `example_001`. Then advance one example at a time in that order.
- Done for an example = C-model `./build.sh && ./run.sh` exit 0 **and** `./build_qemu.sh && ./run_qemu.sh` exit 0.
- No documented skip as success. Fix qemu-ajit and/or cortos2 in-tree until both pass.

## C-model still works
- kind: end-user-interface
- After cortos2 changes, every in-scope example still `./build.sh && ./run.sh` exit 0.
- If C-model is broken before or during work, fix it in this activity.

## Modular cortos2 qemu path
- kind: internal-behavior
- QEMU device map, link rules, output dir, and run helper live in a new cortos2 module, same layering as CoRTOS v1 `targets`/`qemu`. No scattered `if qemu` in the C-model path.

## Doc ledger
- kind: end-user-interface
- `docs/cortos2-on-qemu-ajit.md` has setup, link/UART notes, and per-example pass ledger for all eleven.
- Do not fold into `docs/cortos-on-qemu-ajit.md` (v1 stays). Optional one-line cross-link.
