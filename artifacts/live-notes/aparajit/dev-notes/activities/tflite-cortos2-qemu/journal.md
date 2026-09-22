# Journal

## TFLite Micro on cortos2 (Active → Complete)

Shipped cortos2 yaml hooks `ExtraCc`, `ExtraIncludes`, and `ExtraLibDirs`, plus `os/rtos/cortos2/examples/tflite/` for `hello_world`, `micro_speech`, sixteen kernels, and `person_detection`. They link the existing `libtensorflow-microlite.a`. `hello_world` starts at `main` and passes qemu and the C-model. Every `TEST()` suite starts at `tflite_cortos_start` in `cxxstub.cc`.

Constructor pointers sit in `.rodata` after `ALIGN(4)`. Without that alignment qemu hangs with no UART. qemu widens RAM to the 128MiB guest only when NCRAM does not fit the yaml RAM (`example_001`); the C-model size stays as written.

How to run and the pass table: `repos/ajit-toolchain/docs/cortos2-on-qemu-ajit.md`. `docs/tflite-on-qemu-ajit.md` points at that section. C-model stays skipped for `micro_speech`, the kernels, and `person_detection`. `hello_world` C-model still prints `Thread 0 entered ERROR MODE` before the pass line; `./run.sh` exits 0.

E2e exit 0: `example_001` qemu, `hello_world` qemu and C-model, `micro_speech` (6), sixteen kernels (21, 13, 13, 13, 16, 25, 11, 6, 9, 8, 15, 8, 4, 19, 3, 44), `person_detection` (1).
