# Requirement Definition

## UART pass contract
- kind: end-user-interface
- Each assigned run script exits 0.
- Every non-comment line of that example's `expected_uart.txt` appears on UART.
- Required line: `~~~ALL TESTS PASSED~~~`.
- Fail: missing string, crash, non-zero exit.

## Example tree and scripts
- kind: end-user-interface
- Parent: `os/rtos/cortos2/examples/tflite/`.
- Children: `hello_world/`, `micro_speech/`, `kernels/<name>/`, `person_detection/`.
- Kernel names: `conv`, `depthwise_conv`, `fully_connected`, `softmax`, `add`, `pooling`, `pad`, `activations`, `mul`, `concatenation`, `sub`, `reshape`, `squeeze`, `quantize`, `dequantize`, `reduce`.
- Scripts: `build_qemu.sh` / `run_qemu.sh` for every child. `hello_world` also has `build.sh` / `run.sh`.
- Coverage section in `docs/cortos2-on-qemu-ajit.md`, including the `micro_speech` C-model skip.

## cortos2 C++ link hooks
- kind: software-interface
- yaml `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs`. Paths relative to `$AJIT_HOME`.
- `ExtraCc` compiles with `sparc-linux-g++ -S -fno-pic`, then the existing assembler.
- Linker keeps `.init_array` and `.ctors`.
- `CortosInitCalls` names an unmangled symbol: `main` or `tflite_cortos_start`.
- Project dir contains at least one `.c`.
- Do not flatten the TFLM tree into the example dir.
- Compile `*_test.cc` and model blobs from the submodule.

## Reuse the shipped microlite archive
- kind: software-interface
- Link `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`.
- Submodule stays `$AJIT_HOME/tflite-micro` on `sparc-ajit`.
- Edit that fork only when a selected test cannot link or fails on a SPARC bug.
- `DebugLog` stays the UART path inside the archive.

## Selected tests and step matrix
- kind: internal-behavior
- Order: `hello_world` (qemu + C-model); `micro_speech` (qemu, C-model skip); nine kernels `conv`, `depthwise_conv`, `fully_connected`, `softmax`, `add`, `pooling`, `pad`, `activations`, `mul` (qemu); then `concatenation`, `sub`, `reshape`, `squeeze`, `quantize`, `dequantize`, `reduce` (qemu); `person_detection` (qemu).
- Finish one example's assigned steps before the next.
- Bare-metal is out. Kernel C-model and `micro_speech` C-model are not required to complete.

## Stock cortos2 examples stay green
- kind: software-interface
- Do not change behavior of `example_001`–`example_320`.
- `example_001` `./build_qemu.sh && ./run_qemu.sh` is the regression check.
- A yaml without `ExtraCc` stays on the existing C-only build.
