# TFLite Micro on CoRTOS and qemu-ajit

| Key | Value |
|---|---|
| status | Complete |
| slug | tflite-cortos-qemu |
| branch | none |
| ticket | none |
| notes | Shipped qemu-ajit + CoRTOS TFLM examples; micro_speech C-model skip |

# Goal

Selected TFLite Micro tests build as SPARC V8 32-bit ELFs, print `~~~ALL TESTS PASSED~~~` on UART, and exit 0 on qemu-ajit (bare-metal then CoRTOS). Small cases also run on the C model + CoRTOS. TFLM lives in `ajit-toolchain` as submodule `tflite-micro` on `bb-dev-blocks/tflite-micro` `sparc-ajit`.

# Scope

In: pin `tflite-micro` under `repos/ajit-toolchain`; add TFLM `TARGET=ajit`; thin CoRTOS projects under `os/rtos/cortos/examples/tflite/`; per-example loop bare-metal qemu-ajit → CoRTOS qemu-ajit → C model when assigned. Tests: `hello_world` (steps 2+3+4); `micro_speech` (2+3, 4 unless skip); nine kernel tests (2+3); `person_detection` (2+3). Kernel C-model is a later user call, not required to close.

Out: FPGA, NuttX, ResNet50, qemu-sparc64-user CI as a gate, optimized SPARC kernels, training/converting models, aparajit `repos/tensorflow/`, aparajit-root docker, changing existing CoRTOS `example_001`–`sort` behavior, qemu-ajit CPU/UART map unless a test is blocked.

# Background and Special Notes

- Sibling of `cortos-qemu`. Darwin `ajit_build_dev` bind-mounts only `repos/ajit-toolchain`.
- `sparc-ajit` started as cherry-pick of `dd4f0b4b` onto fork `main` (`ee368b2a`); tip now `dcfebc4f` (`TARGET=ajit`).
- Push org remotes via `git@github-bb:`. Default `github.com` key is read-only for this account.
- SPARC g++ emits `.ctors`; `TEST()` registrars need that walk or UART can print `ALL TESTS PASSED` with 0 tests.

# Current Design

Shipped:

- Submodule `$AJIT_HOME/tflite-micro` → `bb-dev-blocks/tflite-micro` `sparc-ajit` `dcfebc4f`.
- `TARGET=ajit`: SPARC V8 32-bit `libtensorflow-microlite.a`, UART `DebugLog` TX `0xFFFF3204`. CoRTOS links the `.a` only.
- CoRTOS yaml `ExtraCc` / `ExtraIncludes` / `ExtraLibDirs`. g++ `-S -fno-pic` then existing `-s`. Linker keeps `.init_array` and `.ctors`.
- Bare-metal crt0 calls `tflite_call_ctors` then `main`. `TEST()` suites: `CortosInitCalls: tflite_cortos_start`. hello_world: `main`.
- Tree: `os/rtos/cortos/examples/tflite/{hello_world,micro_speech,kernels/<name>,person_detection}/` plus shared `crt0.S`, `cxxstub.cc`, `LinkerScript.qemu.txt`, `run_qemu_elf.py`.
- Coverage/skips: `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md`.
- Pins: ajit-toolchain `469bdeace` (`marshal_updates`); aparajit `0e16916`.

Must not break:

- `example_001` `./build_qemu.sh && ./run_qemu.sh` UART `Hello There`.
- `example_{001,050,100,150,250}` C-model `./build.sh && ./run.sh`.
- Existing CoRTOS example scripts and UART maps.

# Current Plan

Shipped as planned. Kernel C-model left optional.

# Milestones

1. [x] Fork `sparc-ajit` holds SPARC TFLM work
   - tests:
     - remote branch exists and contains fork `main` plus SPARC files
   - evidence:
     - `git ls-remote git@github.com:bb-dev-blocks/tflite-micro.git refs/heads/sparc-ajit` → `dcfebc4f1bf60234c149c83432a0c43a6a33d6b0`

2. [x] `tflite-micro` submodule in `ajit-toolchain` + `TARGET=ajit` lib
   - tests:
     - submodule tracks `sparc-ajit`; `make TARGET=ajit TARGET_ARCH=sparc microlite` yields SPARC ELF32 `.a`
   - evidence:
     - `git -C repos/ajit-toolchain/tflite-micro rev-parse HEAD` → `dcfebc4f1bf60234c149c83432a0c43a6a33d6b0`
     - inside `ajit_build_dev`: `source ./set_ajit_home`; `source docker/ajit_build/ajit_env`; `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite`; `sparc-linux-readelf -h gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a` Machine Sparc

3. [x] Shared `examples/tflite` infra
   - tests:
     - ExtraCc + microlite `.a`; `example_001` qemu green
   - evidence:
     - `cd os/rtos/cortos/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`

4. [x] `hello_world` steps 2, 3, 4
   - tests:
     - UART `~~~ALL TESTS PASSED~~~` bare-metal qemu, CoRTOS qemu, C-model
   - evidence:
     - `cd os/rtos/cortos/examples/tflite/hello_world && ./build_baremetal_qemu.sh && ./run_baremetal_qemu.sh`
     - `./build_qemu.sh && ./run_qemu.sh`
     - `./build.sh && ./run.sh`

5. [x] `micro_speech` steps 2, 3 (4 or skip)
   - tests:
     - UART pass on 2 and 3; step 4 skip in docs
   - evidence:
     - `cd os/rtos/cortos/examples/tflite/micro_speech && ./build_baremetal_qemu.sh && ./run_baremetal_qemu.sh`
     - `./build_qemu.sh && ./run_qemu.sh`
     - skip: `docs/cortos-on-qemu-ajit.md`

6. [x] Nine kernel tests steps 2 and 3
   - tests:
     - `conv`, `depthwise_conv`, `fully_connected`, `softmax`, `add`, `pooling`, `pad`, `activations`, `mul`: UART pass bare-metal and CoRTOS qemu
   - evidence:
     - for `k` in that list: `cd os/rtos/cortos/examples/tflite/kernels/$k && ./build_baremetal_qemu.sh && ./run_baremetal_qemu.sh && ./build_qemu.sh && ./run_qemu.sh`

7. [x] `person_detection` steps 2 and 3
   - tests:
     - UART pass bare-metal and CoRTOS qemu
   - evidence:
     - `cd os/rtos/cortos/examples/tflite/person_detection && ./build_baremetal_qemu.sh && ./run_baremetal_qemu.sh`
     - `./build_qemu.sh && ./run_qemu.sh`

8. [x] Goal e2e: all assigned scripts still pass
   - tests:
     - hello_world 2+3+4, micro_speech 2+3, nine kernels 2+3, person_detection 2+3, `example_001` qemu
   - evidence:
     - same commands as milestones 4–7 plus `cd os/rtos/cortos/examples/example_001 && ./run_qemu.sh` (rebuild if needed)

# Next Steps

Minor-fix runway (not open work):

1. Fastest check: `cd os/rtos/cortos/examples/tflite/hello_world && ./run_baremetal_qemu.sh`; `cd os/rtos/cortos/examples/example_001 && ./run_qemu.sh`.
2. Safest first edits: `os/rtos/cortos/examples/tflite/cxxstub.cc` (ctor walk), yaml `ExtraCc` / `CortosInitCalls`, `tflite-micro/.../ajit/debug_log.cc`.
3. Optional later: kernel C-model; retry `micro_speech` C-model; bake `git`/`numpy`/`PIL` into `ajit_build_dev`; C-model UART `0xFFFF3210` if DebugLog must match the C simulator.

Later product work: `resume-work` then Execution gate.

# References

- `derived-from: cortos-qemu` — qemu-ajit + CoRTOS example scripts; TFLite was out of that activity.
- `git@github.com:bb-dev-blocks/tflite-micro.git` `sparc-ajit` `dcfebc4f` — `TARGET=ajit` on cherry-pick `ee368b2a` of `dd4f0b4b`.
- `tflite-micro/notes-for-self/tflm-sparcv8-port-notes.md` — context-only. Hard problem is BE buffers, not SPARC kernels.
- `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md` — qemu RAM/UART, tflite coverage and `micro_speech` C-model skip.
- `repos/tensorflow/tflite-micro` — leftover clone; not an aparajit submodule.
