# Journal

## CoRTOS on qemu-ajit in the toolchain (Active → Complete)

Vendored qemu-ajit source into `repos/ajit-toolchain/qemu-ajit-v1.0/` (no in-tree `build/`), built in `ajit_build_dev` via `setup_qemu.sh`, PATH through `docker/ajit_build/ajit_env`. CoRTOS `--target qemu` links at `0x00100000`, skips MMU, sets ELF entry to `_start`, writes `cortos_build_qemu/`. Default `cortos build` / `./build.sh` / `./run.sh` stay C-model.

Coverage: `001`, `stads`, `050`, `100` (`VALUE_IS: 20`), `150`, `200`, `250` pass `./run_qemu.sh`. `example_sort` skip after 300s with no UART (1MiB dual-thread quicksort on TCG). C-model five still exit 0. Ledger: `docs/cortos-on-qemu-ajit.md`.

Decisions: asr29 `0x50520000|(core<<8)|thread`; `-smp` from yaml; `ta 0` halts one AJIT CPU (`int32_helper.c`) so the second thread can finish; not LEON `TA0_SHUTDOWN`. Pins: toolchain `1fcd563fd` (`marshal_updates`), parent `77ce38c`. Author `bb-dev-blocks`.

Gaps accepted: sort not a qemu pass; qemu RAM not 16MB-aligned; aparajit-root qemu not updated; no push unless asked. `.dev-notes/` stays gitignored globally.
