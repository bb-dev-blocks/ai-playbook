# Conventions

## Edit the toolchain; TFLM only if blocked
- Product and docs: `repos/ajit-toolchain`. Exception: aparajit `docs/resnet50-metrics.md`.
- Do not edit `tflite-micro` unless C-model UART or a trap blocks the run; then `sparc-ajit` on `git@github-bb:bb-dev-blocks/tflite-micro.git`.
- Do not edit `AjitPublicResources` C-model source.
- Do not track `repos/tensorflow/`. Do not edit aparajit `docker/` or root `cortos/`.
- No activity slugs or milestone ids in product code or comments.

## cortos2 ResNet-50 example
- Dir `os/rtos/cortos2/examples/tflite/resnet50/`. Do not edit `os/rtos/cortos/examples/tflite/resnet50/`.
- Commit `.tflite`; gitignore `gen/`; `cortos_build/` and `cortos_build_qemu/` stay generated.
- Arena 64 MiB; yaml RAM 128 MiB; `CortosInitCalls: main`; stack 256 KiB; `FPU: No`.
- Linker script `.bss` input stays `*(.bss) *(.bss.*)`.

## C-model runs
- Default microarchitecture flags from generated `run_cmodel.sh`, minus `-w`. Record flags with each result.
- No full-model ResNet-50 C-model run. Per-layer runs only, cropped by `ROWS`; ≤ ~10 in parallel.
- Full log per run in a file; results via `cmodel_metrics.py`.
- Leftover simulator: `pkill -x ajit_C_system_m`.
- Cycle figures labeled "C-model estimated cycles"; estimates labeled as sampled + extrapolated.

## Layer harness
- Dir `os/rtos/cortos2/examples/tflite/resnet50_layers/`. Uses `../resnet50/resnet50_int8.tflite`; no second model copy.
- Run ops via TFLM `KernelRunner` + `Register_<OP>()` from the linked library; do not copy kernel bodies or call reference headers directly.
- `VARIANT` default `reference`; results only in `results/<variant>.csv`; variants must match `golden_checksums.tsv`.
- `layers.tsv`, `results/*.csv`, `golden_checksums.tsv`, `vector_assumptions.yaml` committed; `logs/`, `gen/`, `cortos_build/` gitignored.
- Vector figures are labeled analytic estimates with their assumptions file.

## qemu path stays green
- Changes must keep ResNet-50 `./build_qemu.sh && ./run_qemu.sh` passing.
- ResNet-50 example is not edited; the anchor uses its unchanged qemu ELF.
- qemu `expected_uart.txt` keys stay invoke ok + top-1 id/label.

## Sub-agents
- Use sub-agents to split work and keep chat context small.
- Sub-agent model: `cursor-grok-4.6-high`. If missing, stop and ask.

## Commits
- No per-milestone commits. User commits when asked.
- Do not commit the aparajit submodule pointer unless asked.
- Author and committer: `bb-dev-blocks` `<basicblocksdevelopers@gmail.com>`.

## Setup, build, test and install notes
- images: `ajit_base:1.0` then `ajit_build_dev:1.0`. `$AJIT_HOME`=`/home/ajit/ajit-toolchain`.
- attach: from `repos/ajit-toolchain/docker/ajit_build_dev`, `./attach_shell.sh`.
- setup: `source ./set_ajit_home`; `source docker/ajit_build/ajit_env`.
- container exec: `docker exec -u $(id -nu) -w /home/ajit/ajit-toolchain ajit_build_dev bash -c '...'` (not `bash -lc`; login shell drops `LD_LIBRARY_PATH`).
- C-model libs: after `ajit_env`, `export LD_LIBRARY_PATH=$AJIT_HOME/ahir/v2/functionLibrary/lib:$AJIT_HOME/ahir/v2/CtestBench/lib:$AJIT_HOME/ahir/v2/pipeHandler/lib:$LD_LIBRARY_PATH`.
- C-model binary: `ajit_C_system_model` (via `ajit_env`); `-I <n>` prints real time + cycle estimate every n instructions. Long runs: drop `-w main.wtrace` (~5.6× faster).
- example_001 C-model: `cd os/rtos/cortos2/examples/example_001 && ./build.sh && ./run.sh`.
- hello_world C-model: `cd os/rtos/cortos2/examples/tflite/hello_world && ./build.sh && ./run.sh`. `run.sh` checks `expected_uart.txt` against stdout only; C-model halt stats go to stderr, so capture with `2>&1 | tee <log>`.
- `build.sh` in existing examples is `cortos2 build "$@"` (default target `cmodel`).
- metrics: `os/rtos/cortos2/examples/tflite/cmodel_metrics.py <log> [expected_uart.txt]`.
- inventory: `cd os/rtos/cortos2/examples/tflite/resnet50_layers && ./inventory.py` (Python `flatbuffers`; schema module found via `$AJIT_HOME/tflite-micro`).
- one layer: `LAYER=<op index> ROWS=<n> ./build.sh && ./run.sh`.
- sweep: `[VARIANT=<v> TFLM_LIB_DIR=<dir>] ./sweep.sh <linearity|kinds|all|list> -j 4`; total: `./estimate.py [--variant v]`; compare: `./compare.py <base> <variant>`.
- vector estimate: `./vector_estimate.py --assumptions vector_assumptions.yaml [--scenario <name>]`.
- qemu anchor: `./qemu_insn.py layers -j 4` and `./qemu_insn.py model` (needs `../resnet50/build_qemu.sh` first; model run takes hours with the plugin).
- ResNet-50 qemu regression: `./build_qemu.sh && ./run_qemu.sh` (timeout 700).
- example_001 qemu regression: `cd os/rtos/cortos2/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`.
- TFLM lib: `cd tflite-micro && make -f tensorflow/lite/micro/tools/make/Makefile TARGET=ajit TARGET_ARCH=sparc microlite`.
