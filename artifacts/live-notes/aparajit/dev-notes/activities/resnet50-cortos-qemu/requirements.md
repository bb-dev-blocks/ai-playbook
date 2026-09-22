# Requirement Definition

## CoRTOS qemu ResNet-50 inference
- kind: end-user-interface
- Int8 ImageNet ResNet-50 **v1**, input 224×224, runs on CoRTOS + qemu-ajit via TFLM.
- UART prints analysis: model size, input/output shape dtype quant, arena size/used, invoke status, timing if available, top-5 ids/labels/scores, top-1 called out.
- Pass: every non-comment line of the chosen `expected_uart` file appears on UART; exit 0.
- Stable required lines: invoke ok, top-1 class id, top-1 label. Timings and extra lines must not be pass keys.
- Default timeout 600s (hopper ~90s after SPARC unaligned-load fix). Do not replace with a smaller network.

## Build-time sample select
- kind: end-user-interface
- Five small public demo JPEGs under `inputs/`.
- User selects at **build** time: `INPUT=<name>` or `./build_qemu.sh <name>`. Default = first manifest row.
- Build embeds **that** JPEG only. No runtime serial menu.

## Example resource folder
- kind: end-user-interface
- Path: `os/rtos/cortos/examples/tflite/resnet50/` in `repos/ajit-toolchain`.
- Holds: CoRTOS app, yaml, build/run scripts, five JPEGs, ImageNet label list, saved int8 `.tflite`, fetch/pin notes.
- After clone (with committed `.tflite`) plus generate-at-build, qemu run is possible without a second model zoo.

## Recipe docs
- kind: end-user-interface
- Folder: `docs/tflite-micro/` (toolchain).
- ResNet-50 runbook + recipe to bring **another** `.tflite` (quantize/embed/yaml RAM/`INPUT=`/qemu/UART). Recipe-only: no second shipped model.
- Coverage row + link in `docs/cortos-on-qemu-ajit.md`.

## Saved model vs derived blobs
- kind: internal-behavior
- Downloaded Qualcomm AI Hub TFLITE w8a8 blob (`resnet50_int8.tflite`) is the only heavy source artifact kept and committed. Do not commit `resnet50-tflite-w8a8.zip`.
- Derived C arrays (model and selected input) are rebuildable; not committed; gitignored.
- Pin: Hugging Face page https://huggingface.co/qualcomm/ResNet50 plus zip URL + sha256 in `model_pin.txt`. Skip ONNX/DLC/QNN from the same Hub page. Fallback: convert from `tf.keras.applications.ResNet50(weights='imagenet')` (quantize + save `.tflite`, no training).

## Host precheck
- kind: internal-behavior
- In `ajit_build_dev`, run the saved `.tflite` on each of the five JPEGs (TFLite or TFLM x86).
- Write manifest: filename → top-1 id + label. Freeze per-input expected UART snippets from that.
- Fast gate before the long qemu run.

## TFLM link and missing ops
- kind: software-interface
- Link existing `$AJIT_HOME/tflite-micro` `TARGET=ajit` `libtensorflow-microlite.a`. App lives in the example dir (not an upstream TFLM `TEST()`).
- `CortosInitCalls: main` (unmangled).
- Missing op order: another int8 ResNet-50 v1 export → enable existing TFLM reference kernel on `sparc-ajit` → **write** a portable TFLM reference kernel on that fork if the op does not exist. No SPARC-optimized kernels unless that is the only way to complete.
- TFLM commits: `bb-dev-blocks/tflite-micro` `sparc-ajit`. Do not track aparajit `repos/tensorflow/tflite-micro`.

## Existing examples stay green
- kind: software-interface
- Do not change `example_001`–`sort` C-model or qemu behavior.
- `example_001` `./build_qemu.sh && ./run_qemu.sh` remains a regression check.
