# Conventions

## Edit the toolchain; TFLM fork only for a blocking op
- Product and docs: `repos/ajit-toolchain`.
- Touch `tflite-micro` (`sparc-ajit`) only for a missing reference kernel, or if the FlatBuffers `memcpy` fix is absent.
- Push `git@github-bb:bb-dev-blocks/tflite-micro.git`.
- Do not add or track `repos/tensorflow/` in aparajit. Do not edit aparajit `docker/` or root `cortos/`.
- Do not put activity slugs or milestone ids in product code or comments.

## cortos2 ResNet-50 tree is self-contained
- App, five JPEGs, labels, scripts, yaml, pin, and `resnet50_int8.tflite` live in `os/rtos/cortos2/examples/tflite/resnet50/`.
- Copy those assets from the CoRTOS v1 example. Do not edit `os/rtos/cortos/examples/tflite/resnet50/`.
- Link `libtensorflow-microlite.a`. Do not copy TFLM kernel bodies into the example.
- Commit the `.tflite`. Gitignore `gen/`. Do not commit the Hub zip.
- `cortos_build_qemu/` stays generated.

## cortos2 C++ extras
- Yaml uses the cortos2 Hardware/Software shape. Paths in `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs` are relative to `$AJIT_HOME`.
- `ExtraCc` is `sparc-linux-g++ -S -fno-pic`, then the cortos2 assembler.
- `CortosInitCalls: main`. Stack 256 KiB. `FPU: No`.
- Project dir includes one `placeholder.c`.
- Start `RAM.SizeInMegaBytes` at 128. Arena in the app stays 64 MiB.
- Raise `QEMU_GUEST_RAM_MIB` and this RAM to 256 only after link or boot shows the image does not fit.
- Linker script `.bss` input is `*(.bss) *(.bss.*)`. `-fdata-sections` objects must be inside the measured `.bss` or the stack lands on them.

## Build-time INPUT
- Five JPEGs in `inputs/`. `INPUT=<name>` or `./build_qemu.sh <name>`.
- Default = first line of `inputs/order.txt`.
- Embed that JPEG only. Build writes `expected_uart.txt` from `expected/<name>.txt`.

## UART contract
- Verbose UART, then a `top1:` line.
- `expected_uart.txt` substrings: invoke ok, top-1 id, top-1 label.
- qemu pass: every non-comment expected line on stdout; exit 0.
- Wait the full `--timeout 700`. A quiet UART is still the invoke.

## Sub-agents
- Use sub-agents to split work and keep this chat's context small.
- Sub-agent model: `cursor-grok-4.6-high`. If that slug is missing, stop and ask. Do not pick another slug silently.

## Commits
- Do not commit per milestone. User commits when asked.
- Do not commit the aparajit submodule pointer unless asked.
- No secrets. Huge `build/` trees stay untracked.
- Author and committer: `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- attach: from `repos/ajit-toolchain/docker/ajit_build_dev`, `./attach_shell.sh`.
- setup: `source ./set_ajit_home`; `source docker/ajit_build/ajit_env`.
- qemu: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` via `ajit_env`.
- TFLM lib: `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite`.
- TFLM host deps (once per container, root): `apt-get install -y python3-numpy python3-pil` plus a TFLite Python runtime if `host_precheck.py` needs it.
- cwd: `os/rtos/cortos2/examples/tflite/resnet50`.
- fetch: `./fetch.sh` (local zip or AI Hub TFLITE w8a8 URL). Pin file `model_pin.txt`.
- host precheck: `./host_precheck.py`.
- qemu: `./build_qemu.sh` then `./run_qemu.sh` (`cortos2 run --target qemu --timeout 700`). Other input: `INPUT=<name> ./build_qemu.sh && ./run_qemu.sh`.
- pass: `expected_uart.txt` substrings on stdout, exit 0.
- regression: `cd os/rtos/cortos2/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`.
- docs: `docs/tflite-micro/resnet50-cortos2.md` (run + w8a8 fit path), row in `docs/cortos2-on-qemu-ajit.md`.
- no C-model run.
