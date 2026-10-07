# repos -- Dev-Guide

Pinned third-party trees and project forks used for AJIT/APARAJIT system software.

## Notes

- From repo root: `git submodule update --init --recursive`.
- Registered in aparajit `.gitmodules`: `ajit-toolchain`, `gauri-tools`, `PX4-Autopilot`. AHIR lives only inside the toolchain: `repos/ajit-toolchain/ahir`.
- Live NuttX tree for qemu-ajit builds: `repos/ajit-toolchain/nuttx` (`git@github.com:bb-dev-blocks/nuttx.git`, branch `ajit_main`) and sibling `repos/ajit-toolchain/apps` (Apache nuttx-apps). Init from the toolchain repo: `git submodule update --init nuttx apps`. Build inside `ajit_build_dev`.
- Live TFLite Micro tree: `repos/ajit-toolchain/tflite-micro` (`bb-dev-blocks/tflite-micro`, branch `sparc-ajit`).
- Custom QEMU for AJIT v1 lives at repo root: `qemu-ajit-v1.0/` (not under `repos/`).

## Artifacts

| Name | Description |
|------|-------------|
| `repos/ajit-toolchain/` | AJIT cross-toolchain (submodule) |
| `repos/ajit-toolchain/ahir/` | AHIR C/VHDL sources (toolchain submodule; `bb-dev-blocks/ahir`, `marshal_updates`); Docker `$AJIT_HOME/ahir` |
| `repos/gauri-tools/` | AJIT processor tools and validation (submodule) |
| `repos/ajit-toolchain/nuttx/` | Live NuttX kernel submodule (`bb-dev-blocks/nuttx`, `ajit_main`) |
| `repos/ajit-toolchain/apps/` | Live nuttx-apps submodule (Apache, sibling of `nuttx/`) |
| `repos/ajit-toolchain/tflite-micro/` | TFLite Micro fork submodule (`bb-dev-blocks/tflite-micro`, `sparc-ajit`); Edge DNN on AJIT/APARAJIT |
| `repos/PX4-Autopilot/` | PX4 Autopilot fork (submodule; `bb-dev-blocks/PX4-Autopilot`, branch `ajit_nuttx_update`). Overall enablement: `docs/px4-on-nuttx-ajit-qemu.md` |
