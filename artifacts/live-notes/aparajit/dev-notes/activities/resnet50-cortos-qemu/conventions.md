# Conventions

## Edit the toolchain; TFLM fork only for kernels
- Product: `repos/ajit-toolchain`. Example + docs live there.
- Touch `tflite-micro` (`sparc-ajit`) only to enable or write a **reference** kernel needed for this `.tflite`, or for SPARC-correctness of TFLM (unaligned FlatBuffers). Push `git@github-bb:bb-dev-blocks/tflite-micro.git`.
- Do not add or track `repos/tensorflow/tflite-micro` in aparajit.
- Do not edit aparajit `docker/`, `aparajit-docker`, or root `cortos/` for this activity.
- Do not put activity labels (slug, milestone ids) in product code or comments.

## ResNet-50 example is the source tree
- App, five JPEGs, labels, scripts, yaml, and the saved `.tflite` live in `os/rtos/cortos/examples/tflite/resnet50/`.
- Do not copy TFLM kernel/test bodies into the example. Link `libtensorflow-microlite.a`.
- Commit the `.tflite`. Do not commit generated model/input `*_data.cc` (or equivalent).
- `cortos_build/` and `cortos_build_qemu/` stay generated.

## CoRTOS C++ extras
- yaml `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs` (paths relative to `$AJIT_HOME`).
- `ExtraCc`: `sparc-linux-g++ -S -fno-pic`; then `compileToSparcUclibc.py -s`.
- Need at least one `.c` in the project dir (placeholder is fine).
- `CortosInitCalls: main` for this app (explicit `main`, not `TEST()`).
- `TotalMemoryInKB` ≥ 131072 (128 MiB). Do not copy person_detection’s 8192.

## Build-time INPUT
- Five JPEGs in `inputs/`. Select with `INPUT=<name>` or `./build_qemu.sh <name>`.
- Default = first row of the host-precheck manifest.
- Embed that JPEG only. Per-input expected UART from the manifest.

## UART contract
- Verbose analysis on UART (shapes, arena, timing, `rank1`–`rank5`, then `top1`).
- `expected_uart` (per INPUT) lists only stable substrings: invoke ok, top-1 id, top-1 label.
- qemu matcher: every non-comment expected line appears on stdout; exit 0.

## Sub-agents
- Use sub-agents to split work, finish faster, and keep the parent context small.
- Parent (this chat) runs in high mode.
- Sub-agents: model `cursor-grok-4.6-medium`. Do not silently pick another slug if that model is unavailable; stop and ask.

## Commits
- Do not commit per milestone. User commits when asked.
- Do not commit the parent aparajit submodule pointer unless asked.
- No secrets. Huge `build/` trees stay untracked.
- Author and committer for this project: `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- cwd: `repos/ajit-toolchain/docker/ajit_base` and `.../ajit_build_dev`. `./build.sh` / `./run.sh` from that dir.
- attach: `./attach_shell.sh` → `docker exec -u $(id -nu) -w /home/ajit/ajit-toolchain -it ajit_build_dev /bin/bash`.
- setup: inside container, `source ./set_ajit_home`; `./setup.sh`; `./setup_qemu.sh`; `source docker/ajit_build/ajit_env`.
- qemu: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` on `PATH` via `ajit_env`.
- TFLM lib: `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite` → `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`.
- TFLM host deps (once per container, as root): `apt-get install -y git python3-numpy python3-pil` plus TensorFlow Lite Python if host precheck uses it. Not baked into the image.
- cwd example: `os/rtos/cortos/examples/tflite/resnet50`.
- fetch: `./fetch.sh`. Model page for later: https://huggingface.co/qualcomm/ResNet50 (TFLITE w8a8 zip only; skip ONNX/DLC/QNN). Unpacks to `resnet50_int8.tflite`. Accepts a local `resnet50-tflite-w8a8.zip` in the example dir or an ancestor (aparajit root). `.tflite` stays and is committed; do not commit the zip.
- host precheck: `./host_precheck.py` (all five JPEGs; writes `inputs/manifest.tsv` and `expected/<name>.txt`).
- generate: `./generate_runtime.py` from `./build_qemu.sh` (objcopy model `.a`, selected input C array, labels C array). Outputs under `gen/`, gitignored.
- test qemu CoRTOS: `./build_qemu.sh` then `./run_qemu.sh` (timeout 600s; hopper finishes in ~90s after the SPARC unaligned-load fix). Other input: `INPUT=<name> ./build_qemu.sh && ./run_qemu.sh`.
- pass: `expected_uart` stable substrings on stdout, exit 0.
- regression: `cd os/rtos/cortos/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`.
- docs: `docs/tflite-micro/` plus coverage row in `docs/cortos-on-qemu-ajit.md`.
- no C-model run for ResNet-50.
