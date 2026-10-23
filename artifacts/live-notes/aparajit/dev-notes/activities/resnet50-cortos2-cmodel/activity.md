# ResNet-50 on cortos2 and the Ajit C-model: cycles and metrics

| Key | Value |
|---|---|
| status | Active |
| slug | resnet50-cortos2-cmodel |
| branch | none |
| ticket | none |
| notes | replan: full C-model run projected 7–11 days; switch to per-layer sampled measurement + topology-based total |

# Goal

Fix a C-model metric set from cortos2 `example_001` and `examples/tflite/hello_world` on `ajit_C_system_model`. Estimate C-model cycles for one int8 ResNet-50 inference without a full-model run: inventory the 82 ops of the committed `.tflite`, measure every distinct layer configuration on the C-model through a new `resnet50_layers` harness (real weights, output cropped to `ROWS`, ops run via TFLM kernel registrations (`LayerRunner`) so any kernel library variant can be measured), verify crop linearity, sum by topology, and cross-check against an exact qemu `libinsn` instruction count. Harness is reusable for future kernel variants (golden checksums, per-variant results, `compare.py`). Last step: analytic ISA v2 vector-instruction speedup estimate from stated assumptions (`vector_estimate.py`). Docs: aparajit `docs/resnet50-metrics.md` gets the measured estimate plus the detailed process and logic; new `docs/cortos2-cmodel-metrics.md`; `resnet50_layers/README.md` (run specific layers); C-model note in `docs/tflite-micro/resnet50-cortos2.md`; new `docs/tflite-on-cortos2.md` run guide (qemu + C-model) for all cortos2 TFLM examples. qemu ResNet-50, `example_001`, `hello_world` stay green.

# Scope

In: `repos/ajit-toolchain` cortos2 examples and docs, plus aparajit `docs/resnet50-metrics.md`. Discovery on `example_001` and tflite `hello_world` (done). Shared parser `os/rtos/cortos2/examples/tflite/cmodel_metrics.py`. Host inventory script over `resnet50_int8.tflite`. New `os/rtos/cortos2/examples/tflite/resnet50_layers/` (build-time `LAYER=`, `ROWS=`, `VARIANT=`, `TFLM_LIB_DIR=`; ops via `KernelRunner` on real weights; golden checksums; sweep, estimate, compare, vector-estimate scripts; README). qemu instruction anchor on the unchanged ResNet-50 ELF (`libinsn` + `libstoptrigger`). C-model sweep: 23 conv configurations at `ROWS=1`, linearity at `ROWS=1,2,4` for two, FC and 9 non-MAC op kinds cropped; ~4 parallel runs (each C-model process uses ~3.5 cores). Topology sum and 15% cross-check gate. This is the first measured performance baseline of a real CNN on Ajit; later optimization work measures against it.

Out: full-model ResNet-50 C-model run, writing vector or other optimized kernels (vector speedup is estimated only; ISA v2 SIMD is not in the C-model or FPGA prototype), optimizations (SPARC kernels, cache/branch-predictor tuning, memory layout), C-model source edits or speedups, FPGA, other models, TFLM kernel changes, CoRTOS v1 tree `os/rtos/cortos/`, `repos/tensorflow/`. Do not change `example_001`–`example_320` source behavior.

# Background and Special Notes

- cortos2 C-model target: `cortos2 build --target cmodel` (default), build dir `cortos_build/`, MMU on. `cortos2 run` is qemu-only; C-model runs via `./run.sh` → `cortos_build/run_cmodel.sh` → `ajit_C_system_model -R 123456 -n -t -i -m main.mmap.remapped -w main.wtrace [-r results -l log]`.
- C-model source: `AjitPublicResources/processor/64bit/C_multi_core_multi_thread`. Halt prints (stderr): `number-of-instructions-executed`, `number-of-traps`, `number-of-cti`/`mispredicts`, `cycle-count estimate`, branch-predictor + RAS dumps, MMU stats, ICACHE/DCACHE stats. App UART on stdout.
- C-model flags: `-I <n>` progress (real time + cycle estimate every n instr), `-s <pc>` stat trigger PC, `-D/-N/-A/-Q/-L/-b` cache/bp sizes (defaults 512/512/1/1/0/16).
- `__ajit_get_clock_time()` reads `%asr30`/`%asr31`; C-model returns `getCycleEstimate()`. cortos2 wraps it as `cortos_get_clock_time()`; `CORTOS_DEBUG` prints it in brackets.
- `cycle-count estimate` is analytic: instructions + 70×(I+D cache misses) + 250×TLB misses + 16×traps + 4×mispredicts + div/sqrt penalties (`common/include/Ajit_Hardware_Configuration.h`). Not a pipeline simulation.
- MMU-off fetches are uncacheable → counted as misses. Boot dominates small programs; use in-app deltas around the region of interest.
- Env: `bash -c`, not `bash -lc`. C-model needs `ahir/v2/{functionLibrary,CtestBench,pipeHandler}/lib` on `LD_LIBRARY_PATH` (`ahir_release/functionLibrary/lib` missing). TFLM `DebugLog` works on the C-model as built.
- Measured: `example_001` 65203 instr, 4499929 cycles, ~8 s. `hello_world` pass, 1537051 instr, 11004745 cycles, 195 s. Compute loop 20054972 instr, loop cycles 20000102 (CPI 1.0).
- Speed: ~12k instr/s with `-w main.wtrace`; ~68k instr/s without. Full ResNet-50 ~41e9–62e9 instr → 7–11 days; reason for replan.
- Model: 82 ops, 4089184256 MACs. 53 `CONV_2D` (99.95% of MACs) in 23 configurations; largest 3×3 256→256 at 14×14 (5 ops, 578M MACs). 1 FC 2048→1000. Non-MAC: 16 ADD, 4 PAD, 2 CONCATENATION, 2 QUANTIZE, SUB, MUL, MAX_POOL_2D, MEAN.
- TFLM kernels build at `-O2` (`KERNEL_OPTIMIZATION_LEVEL`); cortos2 `OptimizationLevel: 2` matches. Reference kernels are header templates compiled into the app.
- Host tools: Python `flatbuffers` 25.12.19 and `tflite-micro/tensorflow/lite/python/schema_py_generated.py` parse the model. No TF / `tflite_runtime` in the container.
- qemu-ajit 10.0.94 has `-plugin`; `libinsn.so` and `contrib/plugins/libstoptrigger.so` work on SPARC.
- Container: 14 CPUs.
- TFLM `KernelRunner` symbols are in the ajit `libtensorflow-microlite.a`, but its arena is 10000 B (too small for 2048-channel convs); harness uses its own `LayerRunner` (same API path, 256 KB arena).
- cortos2 `ExtraCc` files compile without `-O` (effectively `-O0`); harness glue only, kernels come from the `-O2` library.
- Op 8 (1×1 64→64, 1 row): ~93 cycles/MAC in invoke on the C-model; app boot + Prepare + fill ≈ 100M of 124.6M program cycles. qemu reads `%asr30` without trapping (value not a cycle model).
- Big-endian: `.tflite` constants are little-endian; TFLM's allocator byte-swaps them on SPARC (`micro_allocator.cc`). Harness bypasses it, so `generate_layer.py` swaps multi-byte constants (bias, paddings). PAD failed Prepare before the fix.
- Full-range random int8 input saturates 65–80% of conv outputs (checksum blind to bias); fill is now zero point + 0..31 (int8), [-2, 2) (float). All 40 representative ops pass on qemu.
- cortos2 sized the data region as `.rodata + .data`, missing `.got` + alignment; FC (2 MB weights) hit `.bss overlaps .got`. Fixed in `program.py` (`+ .got + 16`).
- C-model process uses ~3.5 host cores (spinning threads) → sweep `-j 4` on 14 cores.
- Reference conv: ~80–83 instructions/MAC (qemu, exact), invoke CPI ~1.12 → ~91–93 cycles/MAC. Linearity op 9: 93.48 / 93.63 / 93.47 cycles/MAC at ROWS 1/2/4; qemu insns exactly linear. Topology sum of qemu insns ≈ 3.22e11 (CONV_2D 99.6%).
- Op 9 stall split (ROWS 4 − ROWS 1 deltas): mispredicts 52%, I$ 32%, D$ 13%, TLB ~0.
- `libinsn` alone counts until kill (post-program spin); `libstoptrigger.so,addr=` gives exact PC-bounded counts. ResNet-50 instructions before Invoke: 414495589.
- ISA v2 SIMD (`docs/arch/isa_v2.txt`): `VADDD8/16/32`, `VSUBD8/16/32`, `VUMULD8/16/32`, `VSMULD8/16/32` on 64-bit operands. Not implemented in the C-model or FPGA prototype; vector kernels expected months out.
- ResNet-50 example: yaml RAM `0x40000000` 128 MiB, arena 64 MiB, `.rodata` ~25 MiB, `objcopy` blob `gen/libmodelblob.a`. Paper estimate (aparajit `docs/resnet50-metrics.md`): ~61e9 cycles (49e9–82e9).

# Current Design

- Parser `cmodel_metrics.py <log> [expected_uart.txt]`: halt stats, cache/TLB, harness `layer:`/`cycles:` lines, wall-clock; table or CSV row; non-zero exit on missing expected line or stats block.
- Inventory `resnet50_layers/inventory.py`: parse `.tflite` → `layers.tsv` (op index, kind, shapes, stride, padding, MACs/elements, config id, representative flag). Committed.
- Harness `resnet50_layers/`: `generate_layer.py` reads op `LAYER` from `../resnet50/resnet50_int8.tflite` on the host and writes `gen/layer_params.h` (cropped tensor descs, quant, options) + `gen/liblayerblob.a` (cropped constants only). Build-time `LAYER`, `ROWS`, `VARIANT`, `TFLM_LIB_DIR`, `TARGET`; each job in `work/<variant>/L<layer>_R<rows>/` (parallel-safe). App builds `TfLiteTensor`s (real weights/bias; deterministic LCG int8 input), runs `Register_<OP>()` via `LayerRunner`; `cortos_get_clock_time()` around `Invoke`. Prints `layer:` (op, kind, rows, work, unit), `invoke:`, `cycles:`, `checksum:` (FNV-1a of output), `layer_done`.
- Op kinds: CONV_2D, FULLY_CONNECTED, ADD, SUB, MUL, PAD, CONCATENATION, QUANTIZE, MAX_POOL_2D, MEAN.
- `build.sh` → `cortos2 build --target cmodel`; `run.sh` strips `-w` from `run_cmodel.sh`, runs, logs to `logs/<variant>/L<layer>_R<rows>.log`. `sweep.sh all|kinds|linearity|<list> [-j 4]` (heaviest first) → `collect.py` → `results/<variant>.csv` (status, invoke cycles, cycles/unit, checksum, program counters, `tflite-micro` rev, C-model command). Reference `all` also writes `golden_checksums.tsv`; other variants checked against it.
- `qemu_insn.py layers|model`: exact invoke instruction counts on qemu (`libinsn` + `libstoptrigger` at the `call Invoke` and its return) → `results/<variant>-qemu-insn.csv`, `results/qemu-model-insn.txt`.
- `compare.py <base> <variant>`: per-config speedup, topology-weighted total, I$/D$ miss deltas; refuses on checksum mismatch or non-ok row.
- `vector_estimate.py --assumptions vector_assumptions.yaml`: conv/FC insns = MACs × inner insns/MAC + outputs × requant insns + inputs × widen insns; cycles add vector-op extra cycles and the measured reference stall split (mispredict share scales with insns, I$ / D$ shares by scenario factors). Scenarios: scalar-identity, optimized-scalar, vector-conservative / expected / optimistic.
- Linearity: configs (3×3 256→256 at 14×14) and (1×1 64→256 at 56×56) at ROWS 1/2/4; cycles/MAC drift ≤ ~5%.
- Total: `estimate.py` joins `layers.tsv` + `results/<variant>.csv` (+ qemu insn files) → per-op cycles (`results/<variant>-per-op.csv`), per-config / stage / kind, total, weighted invoke CPI.
- Anchor: topology-scaled sum of per-layer qemu invoke insns vs `qemu_insn.py model` (unchanged ResNet-50 ELF); gap gate ±15%. Anchor total = model insns × weighted CPI.
- Docs as in Goal. `docs/resnet50-metrics.md` process section: why full run dropped, C-model speed, cycle formula, inventory, harness, crop + linearity, sweep, sum, anchor, error sources, biases (activation cache footprint in cropped runs; interpreter dispatch excluded; deterministic not real activations).

Must not break: ResNet-50 qemu default build and run, `example_001` qemu + C-model, `hello_world` qemu + C-model.

# Current Plan

1. Discovery on `example_001`, `hello_world` (done).
2. `cmodel_metrics.py` + `docs/cortos2-cmodel-metrics.md`.
3. `inventory.py` → `layers.tsv`.
4. `resnet50_layers` harness for CONV_2D; build/run one conv layer on C-model; README skeleton.
5. Linearity check.
6. Remaining op kinds in the harness.
7. Sweep all configurations; `results.csv`.
8. qemu anchor (`qemu_insn.py`); `estimate.py` total + gate; `compare.py`.
9. Docs: `docs/resnet50-metrics.md` process + results + variant how-to, README, `docs/tflite-micro/resnet50-cortos2.md` note, `docs/tflite-on-cortos2.md`, link in `docs/cortos2-on-qemu-ajit.md`.
10. Vector speedup estimate (`vector_estimate.py`, assumptions file, doc section).
11. e2e regressions.

# Milestones

1. [x] C-model behavior on `example_001`
   - tests:
     - C-model halts after `ta 0`; halt stats block present
     - each stat line mapped to a named metric
     - process exit vs hang recorded (exits)
   - evidence:
     - `cd os/rtos/cortos2/examples/example_001 && ./build.sh && ./run.sh 2>&1 | tee /tmp/ex001_cmodel.log`

2. [x] TFLM `hello_world` on the C-model
   - tests:
     - UART visible without `AJIT_UART_TX` rebuild
     - in-app clock tracks halt `cycle-count estimate`
     - instructions/host-second measured (with and without `-w`)
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/hello_world && ./build.sh && ./run.sh 2>&1 | tee /tmp/hello_cmodel.log`

3. [x] Metric set and parser
   - tests:
     - parser prints table for `example_001` and `hello_world` logs
     - missing expected line → non-zero exit
     - missing stats block → non-zero exit
     - `docs/cortos2-cmodel-metrics.md`: each kept metric, meaning, source line; dropped candidates with reason; env notes
   - evidence:
     - `os/rtos/cortos2/examples/tflite/cmodel_metrics.py /tmp/ex001_cmodel.log`
     - `os/rtos/cortos2/examples/tflite/cmodel_metrics.py /tmp/hello_cmodel.log os/rtos/cortos2/examples/tflite/hello_world/expected_uart.txt`

4. [x] Layer inventory
   - tests:
     - 82 ops; total MACs 4089184256; 53 CONV_2D in 23 configs
     - every op has a config id; one representative per config
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50_layers && ./inventory.py && wc -l layers.tsv`

5. [x] Conv harness on the C-model
   - tests:
     - `LAYER=<conv op> ROWS=1 ./build.sh` exit 0
     - run prints `layer:` + `cycles:` + done line; halt stats present
     - cropped MACs printed = expected from `layers.tsv`
     - op runs through `LayerRunner` + `Register_CONV_2D()` (Prepare ok, Invoke ok)
     - output checksum identical on qemu and C-model (same build inputs)
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50_layers && LAYER=8 ROWS=1 ./build.sh && LAYER=8 ROWS=1 ./run.sh` → op 8 (1×1 64→64 at 56×56), work 229376 MAC (= 56·64·64), invoke 21332972 cycles, checksum `62ad61c5`; whole program 20.4M instr, 124.6M cycles, 308 s
     - `TARGET=qemu LAYER=8 ROWS=1 ./build.sh && TARGET=qemu LAYER=8 ROWS=1 ./run.sh` → checksum `62ad61c5`
     - (checksums above predate the endian fix and new fill; op 8 is now `84e9dc0d` on qemu, C-model match rechecked by the sweep)

6. [x] Linearity
   - tests:
     - two configs at ROWS 1/2/4; cycles per MAC drift ≤ ~5% (else method note + adjustment)
   - evidence:
     - `./sweep.sh linearity` → `results/reference-linearity.csv` rows
     - op 9: 93.48 / 93.63 / 93.47 cycles/MAC (0.2%); op 46: 91.43 / 90.97 / 90.54 (1.0%); qemu invoke insns op 9 exactly ×2, ×4
     - op 46 R1 old fill vs new fill: 91.43 vs 91.31 cycles/MAC (0.13%; cost data-independent)

7. [x] All op kinds in the harness
   - tests:
     - FC, ADD, SUB, MUL, PAD, CONCATENATION, QUANTIZE, MAX_POOL_2D, MEAN each build and run cropped
     - elements printed match `layers.tsv`
   - evidence:
     - `./sweep.sh kinds`
     - all 40 representatives (FC, ADD, SUB, MUL, PAD, CONCATENATION, QUANTIZE, MAX_POOL_2D, MEAN + 23 conv) ok on qemu and C-model; checksums identical

8. [x] Full sweep
   - tests:
     - one `results/reference.csv` row per config (23 conv + FC + non-MAC configs)
     - every row has cycles, instructions, cache/TLB, mispredicts, checksum, `tflite-micro` rev, C-model flags
     - `golden_checksums.tsv` written
   - evidence:
     - `./sweep.sh all -j 4` (split into two runs after a queue restart; final `collect.py --write-golden` over all 40) → `results/reference.csv` 40 rows ok, `golden_checksums.tsv`; 29.8 simulator-hours

9. [x] qemu anchor and topology total
   - tests:
     - `libinsn.so` + `libstoptrigger.so` give exact invoke instruction counts on qemu-ajit SPARC (per layer and full model)
     - default ResNet-50 qemu build unchanged (`top1: 457 bow tie`); no edit to the example
     - `estimate.py`: total, per-stage, per-kind, weighted CPI; anchor total; gap ≤ 15% or explained
     - `compare.py reference reference`: all speedups 1.00×; tampered checksum row → refuses (non-zero exit)
   - evidence:
     - `cd os/rtos/cortos2/examples/tflite/resnet50_layers && ./qemu_insn.py layers && ./qemu_insn.py model`
     - `./estimate.py && ./compare.py reference reference`
     - total 369160994069 cycles (93.0 cycles/MAC, CPI 1.146); qemu model 322128699600 insns vs topology sum 322109256264 (−0.006%); anchor 3.6918e11; compare 1.000×; tampered checksum → refused, exit 1; ResNet-50 qemu `top1: 457 bow tie` (105 s)

10. [x] Docs
    - tests:
      - aparajit `docs/resnet50-metrics.md`: measured headline; paper estimate kept; process and logic section; per-config table; reproduction commands; "evaluate a new kernel variant"
      - `resnet50_layers/README.md`: list layers, run one layer, sweep, linearity, estimate, qemu anchor, new variant (`VARIANT=`, `TFLM_LIB_DIR=`, `compare.py`)
      - `docs/cortos2-cmodel-metrics.md` complete
      - `docs/tflite-micro/resnet50-cortos2.md`: C-model note (full run impractical; link harness and metrics doc)
      - `docs/tflite-on-cortos2.md`: qemu steps all examples; C-model verified for `hello_world`, `resnet50_layers`; generic recipe; coverage table; linked from `docs/cortos2-on-qemu-ajit.md`
    - evidence:
      - `for d in $(find os/rtos/cortos2/examples/tflite -name config.yaml -not -path '*/cortos_build*' -not -path '*/work/*' -exec dirname {} \;); do rg -q "${d#os/rtos/cortos2/examples/tflite/}" docs/tflite-on-cortos2.md || echo "missing $d"; done` prints nothing
      - `rg -n "tflite-on-cortos2.md" docs/cortos2-on-qemu-ajit.md`
      - `rg -n "Process|linearity|libinsn" ../../docs/resnet50-metrics.md`

11. [x] Vector-instruction speedup estimate (template)
    - tests:
      - `vector_assumptions.yaml`: every parameter has value + source (`docs/arch/isa_v2.txt`) or rationale; three scenarios
      - `vector_estimate.py` reads `layers.tsv` + `results/reference.csv`; prints per-config and total cycles + speedup per scenario
      - sanity: all-scalar assumptions (lanes=1, no overhead) reproduce the reference total within 1%
      - changing one assumption changes output (template is live)
      - `docs/resnet50-metrics.md` vector section: assumptions table, results, caveats (not in C-model/FPGA)
    - evidence:
      - `cd os/rtos/cortos2/examples/tflite/resnet50_layers && ./vector_estimate.py --assumptions vector_assumptions.yaml`
      - `./vector_estimate.py --assumptions vector_assumptions.yaml --scenario scalar-identity`
      - scalar-identity 1.00×; optimized-scalar 11.7×; vector conservative / expected / optimistic 14.4× / 19.6× / 25.0×; `inner_insns_per_mac` 1.5→2.0 gives 17.6×

12. [x] Goal e2e
    - result: op 46 rebuilt (`VARIANT=e2e`) → invoke cycles 753985128 identical and checksum `f11906f4` identical to `results/reference.csv`; whole-program counters differ by ≤ 86 cycles / 4 instructions (outside Invoke; build path differs); `estimate.py` and `vector_estimate.py` reproduce the doc numbers; regressions green (run after the cortos2 fix)
    - tests:
      - one conv layer rebuilt and run per README; parser row matches `results/reference.csv` exactly (deterministic), checksum matches golden
      - `estimate.py` reproduces the doc total from committed `layers.tsv` + `results/reference.csv`
      - `vector_estimate.py` reproduces the doc vector table
      - ResNet-50 qemu hopper passes; `example_001` qemu + C-model pass; `hello_world` qemu + C-model pass
    - evidence:
      - `cd os/rtos/cortos2/examples/tflite/resnet50_layers && LAYER=<top conv> ROWS=1 ./build.sh && ./run.sh && ./estimate.py && ./vector_estimate.py --assumptions vector_assumptions.yaml`
      - `cd ../resnet50 && ./build_qemu.sh && ./run_qemu.sh`
      - `cd ../hello_world && ./build_qemu.sh && ./run_qemu.sh && ./build.sh && ./run.sh`
      - `cd ../../example_001 && ./build_qemu.sh && ./run_qemu.sh && ./build.sh && ./run.sh`

# Next Steps

1. Milestone 12: e2e rerun of op 46 (`VARIANT=e2e`) must match `results/reference.csv` exactly; `estimate.py` / `vector_estimate.py` reproduce doc numbers; regressions already green after the cortos2 fix.
2. User review; commit only when asked.

# References

- `derived-from: resnet50-cortos2-qemu` — provenance only.
- `docs/resnet50-metrics.md` (aparajit, untracked) — context-only. Paper estimate ~61e9 cycles at ~15 cycles/MAC; lists C-model run and in-app timer as missing measurements.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/targets.py` — context-only. `cmodel` target: MMU on, `cortos_build/`.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/sys/driver.py` — context-only. `cortos2 run` qemu-only; C-model uses `./run.sh`.
- `repos/ajit-toolchain/os/rtos/cortos2/src/cortos2/xfiles/run_sh/run_cmodel.sh.tpl` — context-only. Generated C-model command; always `-w` when not debug.
- `repos/ajit-toolchain/AjitPublicResources/processor/64bit/C_multi_core_multi_thread/testbench/src/Ajit_C_system_testbench.c` — context-only. CLI flags.
- `repos/ajit-toolchain/AjitPublicResources/processor/64bit/C_multi_core_multi_thread/cpu/src/AjitThread.c` — context-only. `getCycleEstimate`, `printThreadStatistics`, ASR 30/31.
- `repos/ajit-toolchain/AjitPublicResources/processor/64bit/C_multi_core_multi_thread/common/include/Ajit_Hardware_Configuration.h` — context-only. Penalty constants.
- `repos/ajit-toolchain/AjitPublicResources/tools/ajit_access_routines/src/ajit_access_routines.c` — context-only. `__ajit_get_clock_time()`.
- `repos/ajit-toolchain/docs/tflite-on-qemu-ajit.md` — context-only. C-model UART block; v1 ResNet-50 C-model skip.
- `repos/ajit-toolchain/os/rtos/cortos2/examples/tflite/` — context-only. 20 examples; all qemu; only `hello_world` has C-model scripts.
- `repos/ajit-toolchain/os/rtos/cortos2/examples/tflite/resnet50/resnet50_main.cc` — context-only. Resolver op set; phase prints.
- `repos/ajit-toolchain/tflite-micro/tensorflow/lite/micro/tools/make/Makefile` — context-only. `KERNEL_OPTIMIZATION_LEVEL := -O2`.
- `repos/ajit-toolchain/build/qemu-ajit-v1.0/tests/tcg/plugins/libinsn.so` — context-only. Instruction-count plugin for the anchor.
- `repos/ajit-toolchain/docs/arch/isa_v2.txt` — context-only. SIMD `VADDD*`/`VSUBD*`/`VUMULD*`/`VSMULD*` on 64-bit operands (byte/half/word lanes); basis for vector assumptions.
- `repos/ajit-toolchain/tflite-micro/tensorflow/lite/micro/kernels/kernel_runner.h` — context-only. Single-op Init/Prepare/Invoke driver used by the harness.
