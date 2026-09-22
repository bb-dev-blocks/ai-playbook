# Design Decisions

## Example under existing tflite CoRTOS tree
- level: design-choice
- chosen: `os/rtos/cortos/examples/tflite/resnet50/` plus docs `docs/tflite-micro/`.
- why: Same build/run pattern as `person_detection`. Recipe docs are generic; they should not live only under the example.
- alternatives:
  - Docs beside the example only: hides the “other models” recipe.
  - New tree outside CoRTOS examples: splits how to run CoRTOS apps.

## Qualcomm AI Hub TFLITE w8a8 is the saved model
- level: design-choice
- chosen: Qualcomm AI Hub ResNet50 **TFLITE w8a8** (`resnet50-tflite-w8a8.zip` → `resnet50.tflite`, committed as `resnet50_int8.tflite`). Hugging Face page: https://huggingface.co/qualcomm/ResNet50. Zip: qai-hub-models resnet50 v0.62.2.
- why: No official TensorFlow int8 ResNet-50 TFLite URL worked (TF Hub returned HTML). This zip is a ready TFL3 blob (~25 MiB), 224×224, 1000-class ImageNet. User supplied the zip; that is the accepted source.
- skip: ONNX, DLC, and QNN packages from the same page — TFLM cannot load them.
- caveats: I/O is **uint8** (weights/activations 8-bit; filename `resnet50_int8.tflite` is the example’s agreed name). torchvision ResNet-50, not Keras Caffe preprocess. Zip `labels.txt` is 1000 synsets (index 0 = tench); we keep TF `ImageNetLabels.txt` (1001 lines) and apply `label_offset=1`. Host precheck may still fail if the graph has a custom / QNN op.
- alternatives:
  - Keras convert on host: blocked here (no numpy in the host Python). Keep as fetch fallback only.
  - Kaggle TF2 SavedModel: not TFLite.

## Commit the `.tflite`, not derived arrays
- level: design-choice
- chosen: Save and commit the Qualcomm TFLITE w8a8 blob as `resnet50_int8.tflite`. Generate model/input C arrays at build; gitignore them. Do not commit the Hub zip.
- why: One heavy source file. `xxd` output is larger and rebuildable. Matches “do not commit derived heavy artifacts”.
- alternatives:
  - Commit C arrays: huge diffs, still derived.
  - Commit nothing heavy, fetch every clone: example folder would not hold the model.

## Build-time `INPUT=` for five JPEGs
- level: design-choice
- chosen: Five committed JPEGs; `INPUT=<name>` at build embeds that file only. Default = first manifest row.
- why: Headless qemu has no useful prompt. Linking all five bloats the ELF.
- alternatives:
  - Runtime menu on UART: does not fit `./run_qemu.sh`.
  - Link all five: RAM and image size for no gain.

## Verbose UART, stable expected lines
- level: design-choice
- chosen: Print shapes, quant, arena, timing, `rank1`–`rank5` with scores, then `top1: id label`. `expected_uart` matches only invoke-ok + top-1 id/label.
- why: Serial log is for analysis. Timing must not flake the qemu matcher.
- alternatives:
  - Match the full dump: flakes on time and score rounding.

## Host precheck then long qemu
- level: major-implementation-detail
- chosen: x86 TFLite/TFLM in `ajit_build_dev` fills the five-image manifest. qemu-ajit CoRTOS is the e2e; timeout 600s (hopper ~90s). No C-model gate.
- why: SPARC TCG + reference kernels on ResNet-50 can take minutes to hours. C-model already too slow on `micro_speech`.
- alternatives:
  - qemu-only: burns 30+ min on a bad `.tflite`.
  - Hard cap then skip: would not meet the Goal.

## Custom `main`, not TFLM `TEST()`
- level: major-implementation-detail
- chosen: C++ `main` in the example dir; `CortosInitCalls: main`; `MicroMutableOpResolver` with Concatenation and Sub (graph needs them).
- why: ResNet-50 is not an upstream TFLM test. `TEST()` needs ctor walk; this app does not.
- alternatives:
  - Add `resnet50_test.cc` to the TFLM submodule: extra fork churn for an AJIT demo.

## Missing-op fallback includes writing a reference kernel
- level: design-choice
- chosen: Try another int8 ResNet-50 v1 export; then register existing TFLM reference ops; then implement a missing portable kernel on `sparc-ajit`.
- why: Goal is to complete ResNet-50, not to drop the model. SPARC-optimized kernels stay out unless that is the only remaining path.
- alternatives:
  - Example-only, missing op = blocker: can leave the Goal unfinished.
  - Swap MobileNet: not ResNet-50.

## Model blob via objcopy, not xxd
- level: implementation-detail
- chosen: `sparc-linux-objcopy -I binary` → `gen/libmodelblob.a`. Input JPEG still becomes a small C array.
- why: int8 ResNet-50 `.tflite` is ~25 MiB; `xxd` C source would be huge and slow to compile through ExtraCc.
- alternatives:
  - ExtraCc `model_data.cc` from `xxd`: works, painful.

## SPARC unaligned FlatBuffers reads
- level: major-implementation-detail
- chosen: TFLM `flatbuffers.patch` `memcpy`s `ReadScalar` / `IndirectHelper::Read` instead of an unaligned `int64` load.
- why: SPARC V8 traps on unaligned 8-byte loads. Vector payload starts 4 bytes after `size`; QUANTIZE prepare hung in `zero_point()->Get(0)` until this patch.
- alternatives:
  - Keep waiting on qemu: never finishes AllocateTensors.
  - Realign the `.tflite` blob: does not fix in-vector offsets.

## Recipe-only docs for other models
- level: design-choice
- chosen: `docs/tflite-micro/` describes the recipe. Only ResNet-50 is a shipped example.
- why: A second model is independently durable work (`create-sibling`).
- alternatives:
  - Ship a second example in this activity: extra scope.
