# TFLite Micro on cortos2 and qemu-ajit

| Key | Value |
|---|---|
| status | Complete |
| slug | tflite-cortos2-qemu |
| branch | none |
| ticket | none |
| notes | |

# Goal

Selected TFLite Micro tests build under cortos2, print `~~~ALL TESTS PASSED~~~` on the UART, and exit 0 on qemu-ajit. `hello_world` also passes on the cortos2 C-model. cortos2 gains yaml hooks to compile test `.cc` files and link the existing `libtensorflow-microlite.a`.

# Scope

Thin projects under `os/rtos/cortos2/examples/tflite/` for `hello_world`, `micro_speech`, sixteen kernel tests, and `person_detection`. cortos2 qemu for every test. C-model for `hello_world` only. Add `ExtraCc`, `ExtraIncludes`, and `ExtraLibDirs`, with `main` or `tflite_cortos_start` as the init symbol. How to run, plus the coverage table, in `docs/cortos2-on-qemu-ajit.md`. Stock cortos2 examples keep their current behavior. `example_001` qemu is the regression check.

Out: bare-metal redo, re-porting `TARGET=ajit`, moving the `tflite-micro` submodule, the ResNet-50 app, FPGA, NuttX, new SPARC kernels, training, aparajit `repos/tensorflow/`, aparajit-root Docker, edits to CoRTOS v1 `examples/tflite/`, behavior changes to cortos2 `example_001`–`example_320` sources. `micro_speech` and kernel C-model stay unrequired.

# Background and Special Notes

- Product tree is `repos/ajit-toolchain`. cortos2 is `os/rtos/cortos2`. `cortos2 build --target qemu` links at `0x00100000` and writes `cortos_build_qemu/`.
- `tflite-micro` stays the toolchain submodule on `sparc-ajit` `dcfebc4f`. Archive: `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`. `DebugLog` writes UART TX `0xFFFF3204`.
- SPARC g++ emits `.ctors`. Constructor pointers inside `.rodata` must be `ALIGN(4)`. An unaligned pointer hangs qemu with no UART. Skipping the ctor walk can print `ALL TESTS PASSED` with zero tests.
- Sized `operator delete` ships in the cortos2 helper. TFLM virtual destructors need it when libstdc++ is not linked.
- qemu guest RAM is 128MiB. If yaml RAM is smaller than its NCRAM, `--target qemu` widens that window to 128MiB before packing. C-model RAM size stays as written. `example_001` is the case (1MB RAM, two 16MB NCRAM regions).
- `hello_world` C-model prints `Thread 0 entered ERROR MODE` before the UART pass line. `./run.sh` still exits 0. Stack is 256KB; 64KB faulted the stack guard.
- Push org remotes via `git@github-bb:`. The default `github.com` key is read-only for this account.

# Current Design

Shipped. cortos2 yaml keys `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs` (paths relative to `$AJIT_HOME`). `ExtraCc` is `sparc-linux-g++ -S -fno-pic`, then the existing assembler. Linker script keeps `.init_array` and `.ctors` inside `.rodata` after `. = ALIGN(4);`.

Touched paths:

- `os/rtos/cortos2/src/cortos2/sys/config/soft/software.py` — empty defaults for the three lists.
- `os/rtos/cortos2/src/cortos2/sys/config/config.py` — resolve the yaml keys; qemu NCRAM widen.
- `os/rtos/cortos2/src/cortos2/common/consts.py` — `QEMU_GUEST_RAM_MIB`.
- `os/rtos/cortos2/src/cortos2/sys/qemu.py` — uses that constant.
- `os/rtos/cortos2/src/cortos2/xfiles/build_sh/build.sh.tpl` — compile and link the extra `.cc` files.
- `os/rtos/cortos2/src/cortos2/xfiles/linker/LinkerScript.txt.tpl` — ctor arrays, `ALIGN(4)`.
- `os/rtos/cortos2/examples/tflite/` — `cxxstub.cc`, `hello_world`, `micro_speech`, `kernels/<name>`, `person_detection`.
- `os/rtos/cortos2/examples/.gitignore` — `cortos_build_qemu/`.
- `docs/cortos2-on-qemu-ajit.md` — how to run and coverage. `docs/tflite-on-qemu-ajit.md` points here.

Must not break: a yaml without `ExtraCc` stays C-only. Do not edit `os/rtos/cortos/examples/tflite/`. C-model RAM size is not widened. qemu base stays `0x00100000`.

# Current Plan

Done. Hooks, then `hello_world` both platforms, `micro_speech` qemu, nine kernels, seven resolver kernels, `person_detection`, coverage doc, e2e.

# Milestones

1. [x] cortos2 C++ hooks and `hello_world` both platforms
   - evidence: `example_001` and `hello_world` commands below

2. [x] `micro_speech` cortos2 qemu
   - evidence: `cd os/rtos/cortos2/examples/tflite/micro_speech && ./build_qemu.sh && ./run_qemu.sh`

3. [x] Nine original kernel tests, cortos2 qemu
   - evidence: `conv` `depthwise_conv` `fully_connected` `softmax` `add` `pooling` `pad` `activations` `mul`

4. [x] Seven ResNet-50 resolver kernel tests, cortos2 qemu
   - evidence: `concatenation` `sub` `reshape` `squeeze` `quantize` `dequantize` `reduce`

5. [x] `person_detection` cortos2 qemu
   - evidence: `cd os/rtos/cortos2/examples/tflite/person_detection && ./build_qemu.sh && ./run_qemu.sh`

6. [x] Goal e2e: assigned scripts still pass, coverage doc present
   - tests: `hello_world` qemu + C-model, `micro_speech` qemu, sixteen kernels qemu, `person_detection` qemu, `example_001` qemu. Doc lists each test and the C-model skips.
   - evidence: same commands as milestones 1–5. Last run: all exit 0 with `~~~ALL TESTS PASSED~~~`. Counts: micro_speech 6; kernels 21, 13, 13, 13, 16, 25, 11, 6, 9, 8, 15, 8, 4, 19, 3, 44; person_detection 1.

# Next Steps

1. A failing example: `cd os/rtos/cortos2/examples/tflite/<name> && ./build_qemu.sh && ./run_qemu.sh` before touching shared cortos2 code.
2. Safest edits: that example's `config.yaml`, or `docs/cortos2-on-qemu-ajit.md`. Shared traps live in `LinkerScript.txt.tpl` (`ALIGN(4)`) and `config.py` (qemu NCRAM widen).

# References

- `derived-from: tflite-cortos-qemu`
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/resnet50/resnet50_main.cc` — context-only. Learned: the ResNet-50 resolver adds Concatenation, Sub, Reshape, Squeeze, Quantize, Dequantize, and Mean beyond the nine kernel examples. Mean is `reduce_test.cc`.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/cxxstub.cc` — context-only. Learned: `tflite_cortos_start` walks `.init_array` and `.ctors`, then calls `main`. Sized `operator delete` is required without libstdc++.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/config/soft/projectfiles.py` — context-only. Learned: cortos2 picks up local `.c` only.
- `repos/ajit-toolchain/docs/cortos2-on-qemu-ajit.md` — how to run the cortos2 TFLite examples, and the coverage table.
- `git@github.com:bb-dev-blocks/tflite-micro.git` `sparc-ajit` `dcfebc4f` — existing `TARGET=ajit` pin. Do not move it unless a selected test is blocked.
