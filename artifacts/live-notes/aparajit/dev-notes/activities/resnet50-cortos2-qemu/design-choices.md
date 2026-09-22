# Design Decisions

## Example under cortos2 tflite tree
- level: design-choice
- chosen: `os/rtos/cortos2/examples/tflite/resnet50/` plus a cortos2 section in `docs/tflite-micro/resnet50.md` and a row in `docs/cortos2-on-qemu-ajit.md`.
- why: Same build/run pattern as cortos2 `person_detection` and `hello_world`. Recipe stays with the existing ResNet-50 runbook.
- alternatives:
  - New doc file only: splits the one-model recipe.
  - Live under the CoRTOS v1 dir: that tree is frozen for this activity.

## Qualcomm AI Hub TFLITE w8a8 is the saved model
- level: design-choice
- chosen: Same committed bytes as CoRTOS v1 `resnet50_int8.tflite` (sha256 `1753841ba8ce7ca250456db146d0d4c7035d9c15100ce3d2cb563cebde57eb22`). Page https://huggingface.co/qualcomm/ResNet50. Zip qai-hub-models resnet50 v0.62.2.
- why: This blob already invokes on SPARC TFLM. I/O is uint8, 224×224, 1000-class ImageNet. torchvision preprocess, not Keras Caffe. Label list is TF `ImageNetLabels.txt` (1001 lines) with `label_offset=1`.
- alternatives:
  - New export: repeats a solved model search.
  - ONNX/DLC/QNN from that Hub page: TFLM cannot load them.

## Commit the `.tflite`, not derived arrays
- level: design-choice
- chosen: Copy `resnet50_int8.tflite` into the cortos2 example and commit it. Generate model/input/label C outputs at build; gitignore `gen/`. Do not commit the Hub zip.
- why: Clone can build without a second download. `objcopy` output is rebuildable. Identical bytes dedupe in git.
- alternatives:
  - Relative path into the v1 example: cortos2 run depends on a tree this activity must not edit.
  - Commit C arrays: huge diffs.

## Build-time `INPUT=` for five JPEGs
- level: design-choice
- chosen: Five copied JPEGs. `INPUT=<name>` embeds that file only. Default = first `inputs/order.txt` row (`hopper`).
- why: Headless `cortos2 run` has no prompt. Linking all five bloats the ELF.
- alternatives:
  - UART menu: does not fit `./run_qemu.sh`.
  - Link all five: RAM cost for no gain.

## Verbose UART, stable expected lines
- level: design-choice
- chosen: Print shapes, quant, arena, timing, `rank1`–`rank5`, then `top1: id label`. `expected_uart.txt` lists only invoke-ok and top-1 id+label. Build copies `expected/<name>.txt` to that file.
- why: cortos2 qemu passes on non-comment substrings. Timing and scores flake.
- alternatives:
  - Match the full dump: flakes.

## Host precheck then 700s qemu
- level: major-implementation-detail
- chosen: x86 TFLite/TFLM in `ajit_build_dev` fills the five-image manifest. e2e is cortos2 qemu. `cortos2 run --target qemu --timeout 700`. No C-model gate.
- why: v1 finished near 90s with a 600s cap. 700s matches that budget. C-model is out of scope.
- alternatives:
  - qemu-only: burns the full cap on a bad `.tflite`.
  - Timeout 600: user set the wait at about 700s.

## Custom `main`, not TFLM `TEST()`
- level: major-implementation-detail
- chosen: C++ `main`; `CortosInitCalls: main`; stack 256 KiB; `MicroMutableOpResolver` includes Concatenation and Sub.
- why: This is not an upstream TFLM test. cortos2 `hello_world` already inits `main`. `person_detection` uses `tflite_cortos_start` for `TEST()` ctor walk; this app does not.
- alternatives:
  - `tflite_cortos_start`: pulls ctor walk this app does not need.

## Missing-op fallback includes a reference kernel
- level: design-choice
- chosen: Same graph should not need new kernels. If an op is missing: another int8 ResNet-50 v1 export, then register an existing reference op, then implement a portable kernel on `sparc-ajit`.
- why: Goal is this model on cortos2. SPARC-optimized kernels stay out unless nothing else completes.
- alternatives:
  - Stop at a missing op: Goal unfinished.
  - Swap MobileNet: not ResNet-50.

## Model blob via objcopy, not xxd
- level: implementation-detail
- chosen: `sparc-linux-objcopy -I binary` → `gen/libmodelblob.a`. JPEG and labels stay small C arrays.
- why: Blob is about 25 MiB. An `xxd` C file is slow to compile.
- alternatives:
  - ExtraCc of a giant `model_data.cc`: works, painful.

## Rely on the SPARC FlatBuffers memcpy fix
- level: major-implementation-detail
- chosen: Use the `sparc-ajit` tree whose `flatbuffers.patch` `memcpy`s `ReadScalar` / `IndirectHelper::Read`. Do not re-patch unless this build still traps.
- why: SPARC V8 traps on unaligned 8-byte loads. That patch is what let v1 finish `AllocateTensors`.
- alternatives:
  - New patch without a trap: duplicate work.
  - Realign the `.tflite`: does not fix in-vector offsets.

## Guest RAM 128 MiB, then 256
- level: design-choice
- chosen: Yaml RAM 128 MiB, arena 64 MiB, leave `QEMU_GUEST_RAM_MIB` at 128. Raise both to 256 only if link or boot shows the image does not fit.
- why: Weights about 25 MiB plus a 64 MiB arena fit in 128 MiB on paper. cortos2 `-m` is a shared constant. v1 yaml is 256 MiB because v1 sizes `-m` from `TotalMemoryInKB`.
- alternatives:
  - Raise `-m` up front: changes every cortos2 qemu guest with no evidence.
  - Shrink the arena with no `arena_used` measure: can fail invoke.

## Recipe stays in the existing ResNet-50 doc
- level: design-choice
- chosen: Extend `docs/tflite-micro/resnet50.md`. One coverage row in `docs/cortos2-on-qemu-ajit.md`.
- why: One model, two run paths. A second example model is a later activity.
- alternatives:
  - Cortos2-only doc: hides the shared pin, UART, and `INPUT=` rules.

## Gather `.bss.*` into the cortos2 `.bss` output
- level: major-implementation-detail
- chosen: `LinkerScript.txt.tpl` collects `*(.bss) *(.bss.*)`.
- why: `ExtraCc` passes `-fdata-sections`. The 64 MiB arena was a `.bss.<name>` orphan. The second layout pass measured only `.bss` (about 10 KiB) and placed the stack on top of the arena. After the merge, `.bss` is 67119608 bytes and the stack starts at `0x5a92000`. Hopper `arena_used` was 3107344. Guest RAM stayed 128 MiB.
- alternatives:
  - Arena in `placeholder.c` without `-fdata-sections`: hides the same hole for the next large C++ object.

## Dedicated cortos2 runbook
- level: design-choice
- replaces: Recipe stays in the existing ResNet-50 doc
- chosen: `docs/tflite-micro/resnet50-cortos2.md`, written after qemu passes. It tells how to run, and how the w8a8 blob is carried through TFLite Micro and cortos2 so qemu can execute it. Coverage row in `docs/cortos2-on-qemu-ajit.md` links there.
- why: User asked for a docs file at the end, including the fit path, not only a section in the v1 runbook.
- alternatives:
  - Section inside `resnet50.md` only: does not match the requested file.
