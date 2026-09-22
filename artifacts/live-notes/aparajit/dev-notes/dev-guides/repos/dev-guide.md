# repos -- Dev-Guide

Pinned third-party trees and project forks used for AJIT/APARAJIT system software.

## Notes

- From repo root: `git submodule update --init --recursive`.
- Registered in aparajit `.gitmodules`: `ajit-toolchain`, `gauri-tools`, `ahir`.
- Present locally; submodule wiring still settling: `tensorflow/tflite-micro/`. See each tree’s own README for remotes and branches.
- Live NuttX tree for qemu-ajit builds: `repos/ajit-toolchain/nuttx` (`git@github.com:bb-dev-blocks/nuttx.git`, branch `ajit_main`) and sibling `repos/ajit-toolchain/apps` (Apache nuttx-apps). Init from the toolchain repo: `git submodule update --init nuttx apps`. Build inside `ajit_build_dev`.
- `repos/nuttx/` remains the older super-repo (`nuttx/` + `apps/` submodules). It is not the build tree for the AJIT port.
- `repos/tensorflow/tflite-micro/` — TFLite Micro fork for Sparc/AJIT targets; active work on branch `sparc-ajit`.
- Custom QEMU for AJIT v1 lives at repo root: `qemu-ajit-v1.0/` (not under `repos/`).

## Artifacts

| Name | Description |
|------|-------------|
| `repos/ajit-toolchain/` | AJIT cross-toolchain (submodule) |
| `repos/ahir/` | AHIR C/VHDL sources (submodule); pin `88e85c987bfb8a1eaf6d415bbbd1fe625dde764f`. Nested copy also lives at `repos/ajit-toolchain/ahir` for Docker `$AJIT_HOME`. |
| `repos/gauri-tools/` | AJIT processor tools and validation (submodule) |
| `repos/ajit-toolchain/nuttx/` | Live NuttX kernel submodule (`bb-dev-blocks/nuttx`, `ajit_main`) |
| `repos/ajit-toolchain/apps/` | Live nuttx-apps submodule (Apache, sibling of `nuttx/`) |
| `repos/nuttx/` | Older NuttX + apps super-repo; not the AJIT qemu build tree |
| `repos/nuttx/nuttx/` | Older NuttX kernel checkout |
| `repos/nuttx/apps/` | Older nuttx-apps checkout |
| `repos/tensorflow/tflite-micro/` | TFLite Micro fork; Edge DNN on AJIT/APARAJIT |
