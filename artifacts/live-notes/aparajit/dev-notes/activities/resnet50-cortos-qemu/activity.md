# ResNet-50 on CoRTOS and qemu-ajit via TFLM

| Key | Value |
|---|---|
| status | Paused |
| slug | resnet50-cortos-qemu |
| branch | none |
| ticket | none |
| notes | e2e done, uncommitted. Sibling of Complete tflite-cortos-qemu |

# Goal

Int8 ImageNet ResNet-50 v1 runs on CoRTOS + qemu-ajit through TFLite Micro. UART dumps analysis (shapes, arena, timing, top-5). Pass is stable top-1 id+label for a build-time-selected sample. Example folder holds the saved `.tflite` and five JPEGs. `docs/tflite-micro/` is the recipe for this model and for bringing others. Derived C arrays are not git.

# Scope

In: `repos/ajit-toolchain` only. CoRTOS example `os/rtos/cortos/examples/tflite/resnet50/` (app, yaml, scripts, five JPEGs, saved int8 `.tflite`, fetch if needed). Build-time `INPUT=<name>` embeds that JPEG only. Host/container precheck fills a manifest of expected top-1 for all five. qemu-ajit CoRTOS e2e with 600s timeout (hopper ~90s after SPARC unaligned-load fix). Recipe docs under `docs/tflite-micro/` plus a pointer in `docs/cortos-on-qemu-ajit.md`. Missing TFLM op: other int8 ResNet-50 v1 export, then enable or write a **reference** kernel on `sparc-ajit`.

Out: C-model run, FPGA, NuttX, training, SPARC-optimized kernels, full ImageNet val, bare-metal qemu unless CoRTOS is blocked, a second shipped model, aparajit-root docker / `repos/tensorflow/`, changing `example_001`–`sort` behavior.

# Background and Special Notes

- Parent `tflite-cortos-qemu` shipped `TARGET=ajit` microlite + `examples/tflite/{hello_world,micro_speech,kernels,person_detection}`. ResNet-50 was out of that activity. TFLM has no ResNet-50 in-tree.
- Darwin `ajit_build_dev` bind-mounts `repos/ajit-toolchain`. Product and TFLM fork stay there.
- person_detection yaml is 8 MiB; ResNet-50 int8 weights ~25 MiB plus arena. qemu-ajit already floors `-m` at 128 MiB. Raise CoRTOS `TotalMemoryInKB`.
- SPARC TCG + reference kernels will be slow. Host precheck is the fast gate. Do not swap a smaller net if qemu is slow.
- SPARC g++ `.ctors`: this app uses explicit `main` (`CortosInitCalls: main`), not `TEST()`.

# Current Design

- Example: `os/rtos/cortos/examples/tflite/resnet50/`. Shared tflite crt0/cxxstub only if CoRTOS path needs them; start at `main`.
- Source blob: Qualcomm AI Hub ResNet50 **TFLITE w8a8** (`resnet50-tflite-w8a8.zip` member `resnet50.tflite`), saved and committed as `resnet50_int8.tflite`. Pin records https://huggingface.co/qualcomm/ResNet50 plus zip URL and sha256. Do not use ONNX/DLC/QNN from that Hub page. Keras convert stays fetch fallback only.
- Derived (gitignore, rebuild): `xxd`/generate of model C array and the selected input array.
- Five small public JPEGs in `inputs/` plus manifest (name → expected top-1 from host precheck). `./build_qemu.sh` / `INPUT=<name>`; default = first manifest row. Convert **that** JPEG only.
- C++ `main`: `MicroMutableOpResolver` (must include Concatenation and Sub); load model; invoke; UART verbose (model size, io shape/dtype/quant, arena, invoke status, timing, `rank1`–`rank5`, then `top1: id label`). `expected_uart.txt` (or per-input file) requires stable top-1 id+label and invoke-ok only.
- Link existing `libtensorflow-microlite.a`. Fork edits: SPARC-safe FlatBuffers scalar reads (`flatbuffers.patch`) plus per-op UART; no new kernels were required.
- Docs: `docs/tflite-micro/` recipe. Coverage row in `docs/cortos-on-qemu-ajit.md`.
- Host precheck: TFLite or TFLM x86 in `ajit_build_dev` on the saved `.tflite` + each JPEG.

Must not break: `example_001` qemu UART `Hello There`; existing tflite example scripts.

# Current Plan

1. Add example dir, five JPEGs, fetch/save `.tflite`, gitignore for derived arrays, docs stub.
2. Build scripts: generate C arrays from `.tflite` + `INPUT`, CoRTOS yaml RAM, `build_qemu.sh` / `run_qemu.sh` timeout 600s.
3. Write `main` + verbose UART; wire ExtraCc / ExtraIncludes / ExtraLibDirs; `CortosInitCalls: main`.
4. Host precheck writes manifest + per-input expected UART snippets.
5. qemu CoRTOS default `INPUT`. If op missing: other export, then enable/write TFLM reference kernel.
6. Finish `docs/tflite-micro/` recipe; add coverage pointer. No second model.

# Milestones

1. [x] Example tree and saved model
   - tests:
     - dir `os/rtos/cortos/examples/tflite/resnet50/` has scripts, yaml stub, `inputs/` with five JPEGs
     - int8 `.tflite` present; sha256 recorded
     - derived `*_data.cc` gitignored / absent from git
     - `docs/tflite-micro/` stub exists
   - evidence:
     - `ls os/rtos/cortos/examples/tflite/resnet50/inputs`
     - `sha256sum` of the `.tflite` vs recorded pin
     - `git check-ignore -v` on generated array paths (or `git status` shows them untracked)

2. [x] Build-time embed and `INPUT=`
   - tests:
     - `INPUT=<a>` generates arrays from `.tflite` and JPEG `a` only
     - `INPUT=<b>` rebuilds from JPEG `b`; ELF not packed with all five images
   - evidence:
     - inside `ajit_build_dev`: `cd os/rtos/cortos/examples/tflite/resnet50 && INPUT=<a> ./build_qemu.sh`
     - same with `INPUT=<b>`
     - generated array files listed; `git status` does not stage them

3. [x] CoRTOS app links microlite
   - tests:
     - SPARC ELF32; yaml `TotalMemoryInKB` ≥ 131072; `CortosInitCalls: main`
     - ExtraCc includes app + generated arrays + `cxxstub` if required
   - evidence:
     - `./build_qemu.sh` (default INPUT) exit 0
     - `sparc-linux-readelf -h cortos_build_qemu/main.elf` Machine Sparc

4. [x] Host precheck manifest for all five
   - tests:
     - each JPEG: top-1 id + label written
     - per-input expected UART snippet has those stable strings
   - evidence:
     - documented host precheck command in `conventions.md` Setup; run it; cat manifest

5. [x] CoRTOS qemu-ajit default INPUT
   - tests:
     - UART contains invoke-ok, top-1 id+label matching manifest, plus verbose fields (top-5, shapes, arena)
     - `expected_uart` stable lines match; exit 0
     - timeout 600s (hopper ~90s after SPARC unaligned-load fix)
   - evidence:
     - `cd os/rtos/cortos/examples/tflite/resnet50 && ./build_qemu.sh && ./run_qemu.sh`
     - skip C-model: `docs/cortos-on-qemu-ajit.md` / `docs/tflite-micro/`

6. [x] Recipe docs
   - tests:
     - `docs/tflite-micro/` runbook for ResNet-50 and recipe for another `.tflite` (no second example tree)
     - `docs/cortos-on-qemu-ajit.md` coverage row + link
   - evidence:
     - files exist; commands in the runbook match the example scripts

7. [x] Goal e2e
   - tests:
     - host precheck all five
     - qemu CoRTOS default INPUT pass (milestone 5)
     - recipe docs present
     - `example_001` qemu still `Hello There`
   - evidence:
     - host precheck command
     - `cd os/rtos/cortos/examples/tflite/resnet50 && ./run_qemu.sh` (rebuild if needed)
     - `cd os/rtos/cortos/examples/example_001 && ./run_qemu.sh`

# Next Steps

1. Commit three trees: `ajit-toolchain` example+docs+`resnet50_int8.tflite`; `tflite-micro` `sparc-ajit` (`flatbuffers.patch` + per-op UART); aparajit `.gitignore` + activity files. Do not commit `gen/`, Hub zip, or `expected_uart.txt`.
2. Write `mark-completed` to close the activity.

# References

- `derived-from: tflite-cortos-qemu` — TFLM `TARGET=ajit`, CoRTOS ExtraCc, qemu scripts; ResNet-50 was out of scope.
- `repos/ajit-toolchain/os/rtos/cortos/examples/tflite/person_detection/` — context-only. Thin yaml + ExtraCc + `expected_uart.txt`; 8 MiB RAM is too small for ResNet-50; this activity uses `main` not `TEST()`.
- `repos/ajit-toolchain/docs/cortos-on-qemu-ajit.md` — context-only. qemu RAM/UART, tflite coverage ledger; add a ResNet-50 row and link `docs/tflite-micro/`.
- `tflite-micro` `xxd -i` / person_detection `training_a_model.md` — context-only. Embed `.tflite` as C array; here arrays are build products, not committed.
