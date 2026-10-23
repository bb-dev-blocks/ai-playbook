# Requirement Definition

## C-model metric set from simple examples
- kind: end-user-interface
- Run cortos2 `example_001` and `examples/tflite/hello_world` on `ajit_C_system_model` before ResNet-50 work.
- Fix the metric list: meaning, print location, collection method.
- Kept metrics: instructions executed, `cycle-count estimate`, CPI, traps, CTI + mispredicts, RAS push/pop/mispredicts, I/D cache accesses/hits/misses, MMU translated accesses / TLB hits, in-app `cortos_get_clock_time()` deltas, simulator wall-clock and speed. Others dropped with a one-line reason.
- Cycles labeled "C-model estimated cycles" (analytic: instructions + miss/trap/mispredict penalties), not silicon cycles.

## ResNet-50 layer inventory
- kind: internal-behavior
- Host script parses the committed `resnet50_int8.tflite` (sha256 `1753841ba8ce7ca250456db146d0d4c7035d9c15100ce3d2cb563cebde57eb22`): per op index, op kind, input/output/filter shapes, stride/padding, MACs (conv/FC) or element count (others).
- Groups ops into distinct configurations; picks one representative op index per configuration.
- Known shape: 82 ops, 4089184256 MACs, 53 `CONV_2D` in 23 conv configurations, 1 `FULLY_CONNECTED`, 28 non-MAC ops (ADD, PAD, CONCATENATION, QUANTIZE, SUB, MUL, MAX_POOL_2D, MEAN).
- Output committed as a table (op index → configuration, MACs/elements).

## Per-layer C-model harness
- kind: end-user-interface
- New example `os/rtos/cortos2/examples/tflite/resnet50_layers/`.
- Build-time `LAYER=<op index>` and `ROWS=<n>` (output rows; input rows derived).
- Reads that op's real weights, bias, quant params, builtin options from the ResNet-50 example's committed `.tflite` (objcopy blob; no second model copy).
- Runs the op through its TFLM registration (`Register_CONV_2D()` etc.) via TFLM `KernelRunner` (Init/Prepare/Invoke); `Invoke` between two `cortos_get_clock_time()` reads. Measures whatever kernels the linked `libtensorflow-microlite.a` contains.
- UART: layer id, kind, shapes, ROWS, MACs or elements in the cropped run, cycles, output checksum; then a fixed done line.
- C-model run without `-w`; halt stats captured to a log.
- README: how to list layers, build and run one layer, run a sweep, read results.

## Linearity check
- kind: internal-behavior
- Two conv configurations (3×3 256→256 at 14×14; 1×1 64→256 at 56×56) at `ROWS=1,2,4`.
- Pass: cycles per MAC within ~5% across ROWS. Otherwise report and adjust method before extrapolating.

## Sweep and topology total
- kind: end-user-interface
- All 23 conv configurations at `ROWS=1`; FC and the 9 non-MAC op kinds at a cropped size.
- Up to ~10 C-model runs in parallel.
- Total estimated cycles = Σ over all 82 ops (op MACs or elements × measured cycles per MAC or element of its configuration).
- Per-stage and per-op-kind breakdown.

## Reusable kernel-variant evaluation
- kind: end-user-interface
- Build-time `VARIANT=<name>` (default `reference`) and `TFLM_LIB_DIR=<path>` (default `tflite-micro/gen/ajit_sparc_default_gcc/lib`).
- Reference sweep stores per-layer output checksums as golden; any later variant must match exactly before its cycles are accepted.
- Results per variant: `results/<variant>.csv`, tagged with `tflite-micro` git rev and C-model flags.
- `compare.py <base> <variant>`: per-config cycle speedup, total estimated cycles delta, CPI and cache deltas.
- README + `docs/resnet50-metrics.md`: "evaluate a new kernel variant" steps for future optimization activities.

## Vector-instruction speedup estimate
- kind: end-user-interface
- Last analytic step; no vector code runs (ISA v2 SIMD `VADDD*`, `VSUBD*`, `VUMULD*`, `VSMULD*` are not in the C-model or FPGA prototype).
- Template script `vector_estimate.py` + parameter file of stated assumptions (lanes per op, cycles per vector instr, load width, widen/pack overhead, cache penalties unchanged, which op kinds vectorize).
- Input: measured `results/reference.csv` + `layers.tsv`. Output: per-config and total estimated cycles, speedup range (conservative / expected / optimistic).
- Each assumption listed with source (`docs/arch/isa_v2.txt`) or rationale. Re-runnable when assumptions change or hardware lands.
- Section in `docs/resnet50-metrics.md`.

## qemu instruction-count anchor
- kind: internal-behavior
- qemu-ajit `-plugin` with `build/qemu-ajit-v1.0/tests/tcg/plugins/libinsn.so`: exact invoke instruction count, from runs stopped (`libstoptrigger.so`) at the `call Invoke` and at its return; same per harness layer.
- Anchor total = invoke instructions × measured weighted CPI from the sweep.
- Pass: anchor total and per-layer sum within 15%; larger gap reported with cause.
- Also report per-layer-sum instructions vs qemu invoke instructions.

## Docs
- kind: end-user-interface
- aparajit `docs/resnet50-metrics.md`: measured-estimate headline replaces paper estimate; paper estimate kept for comparison; **detailed process and logic** (why full run dropped, C-model speed, cycle model formula, inventory, harness, crop + linearity, sweep, topology sum, qemu anchor, error sources, known biases); per-configuration table; reproduction commands; evaluating a new kernel variant; vector speedup estimate with assumptions.
- `docs/cortos2-cmodel-metrics.md` (`repos/ajit-toolchain`, new): metric meanings, collection, `example_001` and `hello_world` numbers, C-model env (libs, no `-w`, speed).
- `os/rtos/cortos2/examples/tflite/resnet50_layers/README.md`: run specific layers.
- `docs/tflite-micro/resnet50-cortos2.md`: short C-model section pointing at the layer harness and metrics doc; why the full run is not practical.

## cortos2 TFLM run guide
- kind: end-user-interface
- New `docs/tflite-on-cortos2.md` in `repos/ajit-toolchain`, beside `docs/tflite-on-qemu-ajit.md`; linked from `docs/cortos2-on-qemu-ajit.md`.
- Covers all cortos2 TFLM examples under `os/rtos/cortos2/examples/tflite/` (`hello_world`, `micro_speech`, `person_detection`, `resnet50`, `resnet50_layers`, 16 `kernels/*`).
- qemu: steps for every example. C-model: verified steps for `hello_world` and `resnet50_layers`; generic recipe; `resnet50` full C-model marked impractical (projected 7–11 days).
- Coverage table: qemu and C-model status per example.

## Existing paths stay green
- kind: software-interface
- cortos2 ResNet-50 qemu `./build_qemu.sh && ./run_qemu.sh` still passes (hopper `top1: 457 bow tie`); the example itself is not edited.
- `example_001` qemu and C-model still pass. `hello_world` qemu and C-model still pass.
- Do not change `example_001`–`example_320` source behavior. Do not edit CoRTOS v1 tree `os/rtos/cortos/`.

## Out of scope
- kind: internal-behavior
- Full-model ResNet-50 C-model run (projected 7–11 days).
- Optimizations: SPARC or vector kernels, cache/branch-predictor tuning, memory layout. (Vector speedup is estimated only.)
- C-model source edits or speedups, FPGA, other models, TFLM kernel changes, `repos/tensorflow/`.
