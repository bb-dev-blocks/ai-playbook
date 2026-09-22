# Journal

## TFLite Micro on CoRTOS qemu-ajit (Active → Complete)

Shipped `TARGET=ajit` microlite + UART `DebugLog`, CoRTOS `ExtraCc`/`ExtraIncludes`/`ExtraLibDirs`, and thin examples under `os/rtos/cortos/examples/tflite/`. Pass string `~~~ALL TESTS PASSED~~~` on qemu-ajit: hello_world 2+3+4, micro_speech 2+3, nine kernels 2+3, person_detection 2+3, plus `example_001` qemu. Pins: tflite-micro `dcfebc4f` (`sparc-ajit`), ajit-toolchain `469bdeace` (`marshal_updates`), aparajit `0e16916`.

Do not flatten TFLM into CoRTOS; link `libtensorflow-microlite.a`. SPARC g++ needs `-fno-pic` and a `.ctors` walk (`tflite_call_ctors` / `tflite_cortos_start`) or `TEST()` prints a false pass with 0 tests. Coverage: `docs/cortos-on-qemu-ajit.md`.

Accepted gaps: micro_speech C-model skip (full CPU, no test UART in 5+ min); kernel C-model not run; TFLM host deps (`git`, numpy, PIL) not in the image.
