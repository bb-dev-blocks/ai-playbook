# Conventions

## Edit the toolchain and the TFLM fork
- Product: `repos/ajit-toolchain` and its `tflite-micro` submodule.
- TFLM commits go to `git@github.com:bb-dev-blocks/tflite-micro.git` on `sparc-ajit`.
- Do not add or track `repos/tensorflow/tflite-micro` in aparajit.
- Do not edit aparajit `docker/`, `aparajit-docker`, or root `cortos/` for this activity.
- Do not put activity labels (slug, milestone ids) in product code or comments.

## Tests stay in the submodule
- Compile `*_test.cc` and model blobs from `tflite-micro`. Do not copy test bodies into example dirs.
- Example dirs: scripts, `config.yaml`, `expected_uart.txt`, trampoline only if required.
- `cortos_build/` and `cortos_build_qemu/` stay generated.

## CoRTOS C++ extras
- yaml `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs` (paths relative to `$AJIT_HOME`).
- `ExtraCc` is compiled with `sparc-linux-g++ -S -fno-pic`; then `compileToSparcUclibc.py -s`.
- Need at least one `.c` in the project dir (placeholder is fine).
- `CortosInitCalls: main` when the test has an explicit `main` (hello_world).
- `CortosInitCalls: tflite_cortos_start` when the test uses `TEST()` / `TF_LITE_MICRO_TESTS_MAIN` (walks `.init_array` and `.ctors`, then `main`). SPARC g++ emits `.ctors`.

## Sub-agents
- Use sub-agents to split work, finish faster, and keep the parent context small.
- Parent (this chat) runs in high mode.
- Sub-agents: model `cursor-grok-4.6-medium`. Do not silently pick another slug if that model is unavailable; stop and ask.

## Commits
- Do not commit per milestone. User commits when asked.
- Do not commit the parent aparajit submodule pointer unless asked.
- No secrets. Huge `build/` trees stay untracked.
- Author and committer for this project: `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT`=`/home/ajit`, `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- cwd: `repos/ajit-toolchain/docker/ajit_base` and `.../ajit_build_dev`. `./build.sh` / `./run.sh` from that dir.
- attach: `./attach_shell.sh` → `docker exec -u $(id -nu) -w /home/ajit/ajit-toolchain -it ajit_build_dev /bin/bash`.
- setup: inside container, `source ./set_ajit_home`; `./setup.sh`; `./setup_qemu.sh`; `source docker/ajit_build/ajit_env`.
- submodule (once, in `repos/ajit-toolchain`): `git submodule add -b sparc-ajit git@github.com:bb-dev-blocks/tflite-micro.git tflite-micro` then `git -C tflite-micro checkout dcfebc4f1bf60234c149c83432a0c43a6a33d6b0`. Fetch via `git@github-bb:bb-dev-blocks/tflite-micro.git` if `github.com` fails.
- qemu: `$AJIT_HOME/build/qemu-ajit-v1.0/qemu-system-sparc` on `PATH` via `ajit_env`.
- TFLM lib: inside `ajit_build_dev`, `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite`. Output `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`.
- TFLM host deps (once per container, as root): `apt-get install -y git python3-numpy python3-pil`. Not baked into the image yet. `make/downloads/` is gitignored; copy from a prior tree or let scripts fetch (needs `git`).
- test C-model: `cd os/rtos/cortos/examples/tflite/<ex> && ./build.sh && ./run.sh` when step 4 is in scope. `pkill -x ajit_C_system_m` if leftovers.
- TFLM push: `git@github-bb:bb-dev-blocks/tflite-micro.git` (org SSH). `github.com` with the default key is read-only for this account.
- test qemu CoRTOS: `cd os/rtos/cortos/examples/tflite/<ex> && ./build_qemu.sh && ./run_qemu.sh`. Kernels: `.../tflite/kernels/<name>/`.
- test qemu bare-metal: `./build_baremetal_qemu.sh && ./run_baremetal_qemu.sh`.
- `TEST()` suites need `CortosInitCalls: tflite_cortos_start` (walks SPARC `.ctors`). hello_world uses explicit `main`.
- person_detection / micro_speech generate cc arrays via `ensure_genfiles.sh` / `TFLITE_GENERATOR_INPUTS`.
- pass: `expected_uart.txt` substrings on stdout, exit 0, including `~~~ALL TESTS PASSED~~~`.
- regression: `cd os/rtos/cortos/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`.
- docs: `docs/cortos-on-qemu-ajit.md` plus tflite coverage/skips as added.
