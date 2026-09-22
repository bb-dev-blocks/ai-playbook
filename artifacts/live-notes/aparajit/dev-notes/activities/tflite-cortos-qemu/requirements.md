# Requirement Definition

## UART pass contract
- kind: end-user-interface
- Each assigned run script exits 0.
- Every non-comment line of that example's `expected_uart.txt` appears on UART.
- Required line: `~~~ALL TESTS PASSED~~~`.
- Fail: missing string, crash, non-zero exit.

## Example tree and scripts
- kind: end-user-interface
- Parent: `os/rtos/cortos/examples/tflite/`.
- Children: `hello_world/`, `micro_speech/`, `kernels/<name>/` for the nine kernels, `person_detection/`.
- Scripts: step 2 `build_baremetal_qemu.sh` / `run_baremetal_qemu.sh`; step 3 `build_qemu.sh` / `run_qemu.sh`; step 4 `build.sh` / `run.sh` when in scope.
- Short doc next to `docs/cortos-on-qemu-ajit.md` (or a section there) for tflite coverage and skips.

## TFLM `TARGET=ajit` library
- kind: software-interface
- Fork `sparc-ajit` adds `TARGET=ajit`: SPARC V8 32-bit AJIT gcc, UART `DebugLog`, `libtensorflow-microlite.a`.
- Bare-metal test ELFs may use that target's link (RAM `0x00100000`).
- CoRTOS does not use TFLM's ELF; it links the same `.a`.
- `sparc_generic` stays in the fork; not a pass gate for this activity.

## CoRTOS C++ link and start symbol
- kind: software-interface
- `cortos build` may compile listed `.cc` and pass extra `-I` / `-l`.
- `CortosInitCalls` names an unmangled symbol (`main` or `extern "C"` trampoline).
- Do not flatten the TFLM tree into the example dir.
- Compile `*_test.cc` and model blobs from the submodule.

## Toolchain submodule pin
- kind: software-interface
- `$AJIT_HOME/tflite-micro` → `git@github.com:bb-dev-blocks/tflite-micro.git` branch `sparc-ajit`.
- Future TFLM/SPARC commits go to that fork. Aparajit does not track `repos/tensorflow/tflite-micro`.

## Selected tests and step matrix
- kind: internal-behavior
- Order: `hello_world` (2+3+4); `micro_speech` (2+3, 4 unless skip); kernels `conv`, `depthwise_conv`, `fully_connected`, `softmax`, `add`, `pooling`, `pad`, `activations`, `mul` (2+3); `person_detection` (2+3).
- Finish one example's assigned steps before the next.
- Kernel C-model is optional at activity end; not required to complete.

## Existing CoRTOS examples stay green
- kind: software-interface
- Do not change behavior of `example_001`–`sort` C-model or qemu scripts.
- `example_001` `./build_qemu.sh && ./run_qemu.sh` remains a regression check.
