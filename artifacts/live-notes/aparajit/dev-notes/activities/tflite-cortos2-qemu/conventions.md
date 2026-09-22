# Conventions

## Product tree
- Edit `repos/ajit-toolchain`: cortos2, new files under `os/rtos/cortos2/examples/tflite/`, and `docs/cortos2-on-qemu-ajit.md`.
- Edit `tflite-micro` (`sparc-ajit`) only when a selected test cannot link or fails on a SPARC bug. Push `git@github-bb:bb-dev-blocks/tflite-micro.git`.
- Do not edit `os/rtos/cortos/examples/tflite/`.
- Do not add or track `repos/tensorflow/tflite-micro` in aparajit.
- Do not edit aparajit `docker/` or root `cortos/`. No activity labels in product code or comments.

## Tests stay in the submodule
- Compile `*_test.cc` and model blobs from `tflite-micro`. Do not copy test bodies into example dirs.
- Example dirs: scripts, `config.yaml`, `expected_uart.txt`, trampoline only if required.
- `cortos_build/` and `cortos_build_qemu/` stay generated.

## cortos2 C++ extras
- yaml `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs`. Paths relative to `$AJIT_HOME`.
- `ExtraCc`: `sparc-linux-g++ -S -fno-pic`, then the existing assemble step.
- Each project dir needs at least one `.c`.
- `hello_world`: `CortosInitCalls: main`.
- `TEST()` suites: `CortosInitCalls: tflite_cortos_start` in `os/rtos/cortos2/examples/tflite/cxxstub.cc` (ctor walk plus sized `operator delete`).

## Commits
- Do not commit per milestone. User commits when asked.
- Do not commit the aparajit submodule pointer unless asked.
- No secrets. Huge `build/` trees stay untracked.
- Author and committer: `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- attach: `repos/ajit-toolchain/docker/ajit_build_dev/attach_shell.sh`.
- setup: inside the container, `source ./set_ajit_home`; `source docker/ajit_build/ajit_env`.
- submodule pin: `git -C tflite-micro rev-parse HEAD` → `dcfebc4f1bf60234c149c83432a0c43a6a33d6b0` on `sparc-ajit`. Fetch via `git@github-bb:bb-dev-blocks/tflite-micro.git` if `github.com` fails.
- TFLM lib (only if the archive is missing): `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite`. Output `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`.
- Host deps for a lib rebuild, once per container as root: `apt-get install -y git python3-numpy python3-pil`.
- test qemu: `cd os/rtos/cortos2/examples/tflite/<ex> && ./build_qemu.sh && ./run_qemu.sh`. Kernels: `.../tflite/kernels/<name>/`.
- test C-model (`hello_world` only): `./build.sh && ./run.sh`. `pkill -x ajit_C_system_m` if leftovers.
- `person_detection` and `micro_speech` generate cc arrays before compile when the genfiles are absent.
- pass: `expected_uart.txt` substrings on stdout, exit 0, including `~~~ALL TESTS PASSED~~~`.
- regression: `cd os/rtos/cortos2/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`.
- docs: add the tflite coverage section to `docs/cortos2-on-qemu-ajit.md`.
