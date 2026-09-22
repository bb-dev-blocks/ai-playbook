# Design Decisions

## Reuse the shipped microlite archive
- level: design-choice
- chosen: Link the existing `TARGET=ajit` `libtensorflow-microlite.a`. Submodule stays `$AJIT_HOME/tflite-micro`, branch `sparc-ajit`, pin `dcfebc4f` unless a selected test is blocked.
- why: That archive and its UART `DebugLog` already exist. This activity is the cortos2 consumer.
- alternatives:
  - New TFLM target or an aparajit `repos/tensorflow` tree: repeats finished work.

## Thin cortos2 examples
- level: design-choice
- chosen: `os/rtos/cortos2/examples/tflite/<test>/` holds scripts, `config.yaml`, `expected_uart.txt`, and a trampoline only if required. Shared helper is `examples/tflite/cxxstub.cc`. Test sources stay in the submodule.
- why: cortos2 projects are flat C. The TFLM tree is C++.
- alternatives:
  - Teach cortos2 to compile the whole TFLM tree: out of scope.

## cortos2 yaml C++ extras
- level: design-choice
- chosen: Add `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs`. `g++ -S -fno-pic`, then the existing assembler. Linker script keeps `.init_array` and `.ctors`.
- why: cortos2 currently lists local `.c` only. Per-example side makefiles would duplicate flags.
- alternatives:
  - One makefile per example outside `cortos2 build`: bypasses the qemu target and the C-model path.

## Unmangled init, ctor walk, sized delete
- level: major-implementation-detail
- chosen: `hello_world` uses `CortosInitCalls: main`. `TEST()` suites use `tflite_cortos_start`, which walks `.init_array` and `.ctors`, then calls `main`. The helper lives under the cortos2 tflite parent and also defines sized `operator delete`.
- why: SPARC g++ emits `.ctors`. An empty walk prints a false pass. Virtual destructors need sized delete when libstdc++ is absent. CoRTOS v1 `examples/tflite/` stays untouched.
- alternatives:
  - Call the v1 `cxxstub.cc` from cortos2: couples the two RTOS trees.

## UART log path stays DebugLog
- level: major-implementation-detail
- chosen: Tests keep TFLM `DebugLog` (UART TX `0xFFFF3204`). The test body does not call cortos2 printf.
- why: Bare-metal and cortos2 then share one log path inside the archive. This activity does not redo bare-metal, and the contract stays the same string.
- alternatives:
  - cortos2 printf on the qemu step: a second log implementation.

## Kernel set adds the ResNet-50 resolver gaps
- level: design-choice
- chosen: Sixteen kernel dirs. The seven added names are `concatenation`, `sub`, `reshape`, `squeeze`, `quantize`, `dequantize`, `reduce` (`Mean` is `reduce_test.cc`). qemu only.
- why: Those ops are on the ResNet-50 resolver and have no kernel example in the nine. The ResNet-50 app stays out.
- alternatives:
  - Only `concatenation` and `sub`: leaves Mean, Reshape, Squeeze, Quantize, and Dequantize untested.

## C-model only for hello_world
- level: design-choice
- chosen: `hello_world` runs `./build.sh && ./run.sh` and the qemu scripts. Every other assigned test is qemu only. `micro_speech` C-model is a documented skip. No bare-metal scripts.
- why: Matches the closed CoRTOS v1 matrix. Bare-metal ELFs already exist under CoRTOS v1.
- alternatives:
  - Repeat bare-metal under cortos2: duplicate scripts.

## Stock-example regression is example_001
- level: implementation-detail
- chosen: Do not edit `example_001`–`example_320`. Runnable check is `example_001` qemu. Full eleven-example rerun is not this activity's e2e.
- why: Hooks are opt-in. The other ten stay green by not changing their sources. Their dual-pass gate belongs to `cortos2-qemu`.
- alternatives:
  - Re-run all eleven here: duplicates that gate.
