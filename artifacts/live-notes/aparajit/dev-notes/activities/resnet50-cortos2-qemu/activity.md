# ResNet-50 on cortos2 and qemu-ajit via TFLM

| Key | Value |
|---|---|
| status | Active |
| slug | resnet50-cortos2-qemu |
| branch | none |
| ticket | none |
| notes | qemu timeout 700s; RAM 128 MiB unless link or boot misses, then 256 |

# Goal

Int8 ImageNet ResNet-50 v1 runs on cortos2 + qemu-ajit through TFLite Micro. UART dumps analysis (shapes, arena, timing, top-5). Pass is stable top-1 id+label for a build-time-selected JPEG. Example folder holds the saved `.tflite` and five JPEGs. `docs/tflite-micro/resnet50-cortos2.md` is the cortos2 runbook: how to run, and how the w8a8 blob is carried through TFLite Micro and cortos2 onto qemu. `docs/cortos2-on-qemu-ajit.md` has a coverage row. Derived C arrays are not git. Script timeout is 700s.

# Scope

In: `repos/ajit-toolchain` only. cortos2 example `os/rtos/cortos2/examples/tflite/resnet50/` (app, yaml, scripts, five JPEGs, saved int8 `.tflite`, fetch pin). Build-time `INPUT=<name>` embeds that JPEG only. Host precheck in `ajit_build_dev` writes a manifest of expected top-1 for all five. qemu-ajit cortos2 e2e uses `cortos2 run --target qemu --timeout 700`. Yaml RAM starts at 128 MiB with a 64 MiB tensor arena. If link or boot shows the image does not fit in the 128 MiB guest, set `QEMU_GUEST_RAM_MIB` and this example's RAM to 256. After the qemu path works, write `docs/tflite-micro/resnet50-cortos2.md` (how to run, and how the w8a8 model is made to fit through TFLite Micro and cortos2 onto qemu). Add a coverage row in `docs/cortos2-on-qemu-ajit.md` that links to it.

Out: edits to `os/rtos/cortos/examples/tflite/resnet50/`, re-porting `TARGET=ajit`, a second model, C-model, FPGA, NuttX, training, SPARC-optimized kernels, full ImageNet val, aparajit-root Docker, `repos/tensorflow/`. Do not change cortos2 `example_001`–`example_320` source behavior. Do not raise qemu `-m` before a fit failure.

# Background and Special Notes

- cortos2 qemu links at `0x00100000`, build dir `cortos_build_qemu/`. `cortos2 run` matches non-comment lines of `expected_uart.txt`. Guest size is `QEMU_GUEST_RAM_MIB` (128) in `os/rtos/cortos2/src/cortos2/common/consts.py`. Yaml RAM above that does not enlarge `-m`.
- Existing cortos2 TFLM examples use `ExtraCc`, `ExtraIncludes`, `ExtraLibDirs`. `hello_world` inits `main`. `person_detection` inits `tflite_cortos_start` and uses 8 MiB RAM.
- CoRTOS v1 ResNet-50 at `os/rtos/cortos/examples/tflite/resnet50/` already runs this graph: Qualcomm TFLITE w8a8 blob, arena 64 MiB, yaml `TotalMemoryInKB: 262144`, `CortosInitCalls: main`, `run_qemu.sh` timeout 600. Measured invoke about 90s after the SPARC unaligned-load fix. v1 tree stays untouched; copy assets out of it.
- `libtensorflow-microlite.a` is `TARGET=ajit` on `sparc-ajit`. FlatBuffers scalar reads already use `memcpy` via `flatbuffers.patch`. This app does not add kernels.
- Silence on UART until the 700s cap means the invoke is still running.

# Current Design

- Example: `os/rtos/cortos2/examples/tflite/resnet50/`. Copy the v1 `resnet50_int8.tflite`, `model_pin.txt`, five JPEGs, `inputs/order.txt`, `inputs/manifest.tsv` as a starting point, `imagenet_labels.txt`, and the app sources. Rebuild scripts against `cortos2 build` / `cortos2 run`.
- cortos2 yaml shape (Hardware + Software), one core, `FPU: No`, `CortosInitCalls: main`, stack 256 KiB. `RAM.SizeInMegaBytes: 128`. `ExtraCc` lists `resnet50_main.cc`, generated input and labels, and `os/rtos/cortos2/examples/tflite/cxxstub.cc`.
- Arena stays 64 MiB. Model blob: `sparc-linux-objcopy -I binary` into `gen/libmodelblob.a`. Selected JPEG and labels become small C arrays under `gen/` (gitignored).
- `CortosInitCalls: main`. `MicroMutableOpResolver` includes Concatenation and Sub. UART: model size, io shape/dtype/quant, arena, invoke status, timing, `rank1`–`rank5`, then `top1: id label`. `./build_qemu.sh` copies `expected/<name>.txt` to `expected_uart.txt`. Matcher requires invoke-ok and top-1 id+label only.
- Link `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`. No TFLM edit unless a missing op blocks this same graph (other int8 ResNet-50 v1 export, then enable or write a reference kernel on `sparc-ajit`).
- Fit: yaml RAM 128 MiB was enough. `.rodata` 26518136 bytes, `.bss` 67119608 bytes (64 MiB arena), stack at `0x5a92000`. `LinkerScript.txt.tpl` gathers `*(.bss) *(.bss.*)` so `-fdata-sections` does not leave the arena outside the measured `.bss`. Hopper `arena_used` was 3107344. `QEMU_GUEST_RAM_MIB` stayed 128. `example_001` qemu still prints `Hello There`.
- Docs, last: `docs/tflite-micro/resnet50-cortos2.md` (run steps plus the w8a8 → TFLM → cortos2 → qemu fit). Coverage row in `docs/cortos2-on-qemu-ajit.md`.

Must not break: cortos2 `example_001` qemu UART. Do not modify the CoRTOS v1 ResNet-50 tree.

# Current Plan

1. Add the cortos2 example dir: copy model, pin, JPEGs, labels; yaml at 128 MiB; `gen/` gitignore; script stubs.
2. `generate_runtime.py` + `build_qemu.sh`: objcopy model, embed `INPUT` only, write `expected_uart.txt`, `cortos2 build --target qemu`.
3. Port `resnet50_main.cc` (`main`, resolver, verbose UART). Wire ExtraCc / ExtraIncludes / ExtraLibDirs to cortos2 `cxxstub.cc`.
4. If link or boot misses RAM, raise guest and yaml RAM to 256 and relink.
5. `host_precheck.py` refreshes manifest and `expected/<name>.txt` for all five.
6. `run_qemu.sh` is `cortos2 run --target qemu --timeout 700` for default `INPUT`.
7. After qemu passes, write `docs/tflite-micro/resnet50-cortos2.md` and the coverage row.

# Milestones

1. [x] Example tree and saved model
   - tests:
     - dir has scripts, yaml, `inputs/` with five JPEGs, `resnet50_int8.tflite`, `model_pin.txt`
     - sha256 matches pin `1753841ba8ce7ca250456db146d0d4c7035d9c15100ce3d2cb563cebde57eb22`
     - `gen/` gitignored
   - evidence:
     - `ls os/rtos/cortos2/examples/tflite/resnet50/inputs`
     - `sha256sum os/rtos/cortos2/examples/tflite/resnet50/resnet50_int8.tflite`
     - `git check-ignore -v os/rtos/cortos2/examples/tflite/resnet50/gen/input_data.cc`

2. [x] Build-time embed and `INPUT=`
   - tests:
     - `INPUT=<a>` generates model archive plus JPEG `a` only
     - `INPUT=<b>` rebuilds from JPEG `b`; ELF does not contain all five images
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50 && INPUT=hopper ./build_qemu.sh`
     - `INPUT=panda ./build_qemu.sh`
     - `git status` does not stage `gen/`

3. [x] cortos2 app links microlite inside guest RAM
   - tests:
     - SPARC ELF; yaml RAM 128 MiB; `CortosInitCalls: main`; stack 256 KiB
     - link succeeds
     - on fit failure only: `QEMU_GUEST_RAM_MIB` 256, yaml RAM 256, relink succeeds
   - evidence:
     - `./build_qemu.sh` exit 0
     - `sparc-linux-readelf -h cortos_build_qemu/main.elf` Machine Sparc
     - `rg QEMU_GUEST_RAM_MIB os/rtos/cortos2/src/cortos2/common/consts.py`

4. [x] Host precheck manifest for all five
   - tests:
     - each JPEG: top-1 id + label
     - `expected/<name>.txt` has invoke-ok and that top-1
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50 && ./host_precheck.py`
     - `cat inputs/manifest.tsv`

5. [x] cortos2 qemu-ajit default INPUT
   - tests:
     - UART has invoke-ok, top-1 id+label from the manifest, plus verbose fields
     - every non-comment `expected_uart.txt` line appears; exit 0
     - process is allowed the full 700s
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50 && ./build_qemu.sh && ./run_qemu.sh`

6. [x] Recipe docs
   - tests:
     - `docs/tflite-micro/resnet50-cortos2.md` states how to run and how w8a8 fits through TFLM and cortos2 onto qemu (dir, 700s timeout, RAM rule, embed, resolver, FlatBuffers)
     - `docs/cortos2-on-qemu-ajit.md` has a ResNet-50 coverage row and link
   - evidence:
     - those two files contain the cortos2 commands

7. [x] Goal e2e
   - tests:
     - host precheck all five
     - qemu default INPUT pass (milestone 5)
     - docs present (milestone 6)
     - cortos2 `example_001` qemu still passes
   - evidence:
     - `./host_precheck.py`
     - `cd os/rtos/cortos2/examples/tflite/resnet50 && ./run_qemu.sh`
     - `cd os/rtos/cortos2/examples/example_001 && ./build_qemu.sh && ./run_qemu.sh`

# Next Steps

1. User commits when asked. Do not commit `gen/`, `cortos_build_qemu/`, or the Hub zip.
2. `mark-completed` when they want the activity closed. e2e already ran: host precheck, hopper qemu `top1: 457 bow tie`, `example_001` `Hello There`.

# References

- `derived-from: resnet50-cortos-qemu` — provenance only.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/resnet50/` — context-only. On-disk source of the Qualcomm blob, five JPEGs, pin, and `main`. Copy out; do not edit.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/resnet50/model_pin.txt` — context-only. sha256 `1753841ba8ce7ca250456db146d0d4c7035d9c15100ce3d2cb563cebde57eb22`, Hugging Face page https://huggingface.co/qualcomm/ResNet50, zip URL, uint8 224×224 input.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/resnet50/resnet50_main.cc` — context-only. Arena `64 * 1024 * 1024`; UART keys `invoke` and `top1`.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/resnet50/inputs/manifest.tsv` — context-only. Default order starts at hopper, top-1 `457 bow tie` under the v1 precheck.
- `repos/ajit-toolchain/os/rtos/cortos2/examples/tflite/hello_world/config.yaml` — context-only. cortos2 yaml shape; `CortosInitCalls: main`; stack 256 KiB.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/common/consts.py` — context-only. `QEMU_GUEST_RAM_MIB = 128`.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/qemu.py` — context-only. Pass/fail is `expected_uart.txt`; `-m` comes from `QEMU_GUEST_RAM_MIB`.
- `repos/ajit-toolchain/docs/tflite-micro/resnet50.md` — context-only. v1 runbook. cortos2 gets its own file `docs/tflite-micro/resnet50-cortos2.md`.
- `repos/ajit-toolchain/docs/cortos2-on-qemu-ajit.md` — context-only. Coverage table; person_detection row is the pattern for a new ResNet-50 row.
