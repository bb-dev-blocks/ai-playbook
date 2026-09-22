# Requirement Definition

## cortos2 qemu ResNet-50 inference
- kind: end-user-interface
- Int8 ImageNet ResNet-50 v1, input 224×224, runs on cortos2 + qemu-ajit via TFLM.
- UART prints model size, input/output shape dtype quant, arena size/used, invoke status, timing if available, top-5 ids/labels/scores, then top-1.
- Pass: every non-comment line of `expected_uart.txt` appears on UART; exit 0.
- Stable required lines: invoke ok, top-1 class id, top-1 label. Timings and scores are not pass keys.
- Timeout 700s. A quiet UART before that cap means the invoke is still running. Do not replace the model with a smaller network.

## Build-time sample select
- kind: end-user-interface
- Five small public demo JPEGs under `inputs/`.
- Select at build time: `INPUT=<name>` or `./build_qemu.sh <name>`. Default = first `inputs/order.txt` row.
- Build embeds that JPEG only. No runtime serial menu.

## Example resource folder
- kind: end-user-interface
- Path: `os/rtos/cortos2/examples/tflite/resnet50/` in `repos/ajit-toolchain`.
- Holds: cortos2 app, yaml, build/run scripts, five JPEGs, ImageNet label list, saved int8 `.tflite`, fetch/pin notes.
- After clone (with committed `.tflite`) plus generate-at-build, qemu run does not need a second download.

## Recipe docs
- kind: end-user-interface
- File `docs/tflite-micro/resnet50-cortos2.md`, written after the qemu path works.
- How to run: dir, `INPUT=`, host precheck, `./build_qemu.sh`, `./run_qemu.sh`, 700s timeout, RAM rule, pass lines.
- How the w8a8 model is made to fit: Qualcomm TFLITE blob, TFLM `sparc-ajit` link, objcopy into the cortos2 image, op resolver, FlatBuffers unaligned reads, cortos2 section layout inside guest RAM, qemu.
- Coverage row + link in `docs/cortos2-on-qemu-ajit.md`.
- No second shipped model.

## Saved model vs derived blobs
- kind: internal-behavior
- Committed blob is Qualcomm AI Hub ResNet50 TFLITE w8a8, file `resnet50_int8.tflite`.
- Pin: sha256 `1753841ba8ce7ca250456db146d0d4c7035d9c15100ce3d2cb563cebde57eb22`, page https://huggingface.co/qualcomm/ResNet50, zip `resnet50-tflite-w8a8.zip` from qai-hub-models resnet50 v0.62.2. Skip ONNX/DLC/QNN.
- Do not commit the zip. Derived C arrays and `gen/` are gitignored.
- Copy the blob into this example. Do not edit `os/rtos/cortos/examples/tflite/resnet50/`.

## Host precheck
- kind: internal-behavior
- In `ajit_build_dev`, run the saved `.tflite` on each of the five JPEGs (TFLite or TFLM x86).
- Write `inputs/manifest.tsv` (filename, top-1 id, label) and `expected/<name>.txt`.
- Fast gate before the 700s qemu run.

## TFLM link and missing ops
- kind: software-interface
- Link existing `$AJIT_HOME/tflite-micro` `TARGET=ajit` `libtensorflow-microlite.a`.
- App `main` is unmangled. `CortosInitCalls: main`.
- Missing op order: another int8 ResNet-50 v1 export, then enable an existing TFLM reference kernel on `sparc-ajit`, then write a portable reference kernel on that fork. No SPARC-optimized kernels unless that is the only way to finish.
- TFLM commits stay on `bb-dev-blocks/tflite-micro` `sparc-ajit`. Do not track aparajit `repos/tensorflow/`.

## Guest RAM
- kind: software-interface
- Start yaml `RAM.SizeInMegaBytes` at 128. Tensor arena stays 64 MiB.
- `QEMU_GUEST_RAM_MIB` stays 128 unless link or boot shows this image does not fit.
- On that failure only: set `QEMU_GUEST_RAM_MIB` to 256 and this example RAM to 256, then relink.
- Other cortos2 examples keep their own yaml RAM sizes.

## Existing examples stay green
- kind: software-interface
- Do not change cortos2 `example_001`–`example_320` source behavior.
- `example_001` `./build_qemu.sh && ./run_qemu.sh` remains the regression check.
- Do not edit the CoRTOS v1 ResNet-50 example.
