# Design Decisions

## Discover metrics on simple examples first
- level: design-choice
- chosen: `example_001` then `examples/tflite/hello_world` on the C-model; fix the metric list; then ResNet-50.
- why: ResNet-50 C-model run may take hours. Output format, halt behavior, UART mapping, cycle counter semantics are cheap to learn on small runs.
- alternatives:
  - ResNet-50 first: hours lost per wrong assumption.
  - TFLM kernel test (`conv`) too: not needed to learn the metric set.

## Cycles come from the C-model cycle estimate
- level: major-implementation-detail
- chosen: In-app phase timing uses `cortos_get_clock_time()` → `__ajit_get_clock_time()` → `rd %asr30/%asr31`. C-model `getAsrValue` returns `getCycleEstimate()` for those ASRs. Halt stats print the same `cycle-count estimate`.
- why: One counter for phase and total figures; cross-checkable.
- alternatives:
  - Wire TFLM `GetCurrentTimeTicks()`: edits TFLM target; app-local call is enough.
  - `__ajit_sample_thread_performance_counters`: MMIO perf counters; C-model support unverified.

## Default C-model microarchitecture settings
- level: design-choice
- chosen: Keep `run_cmodel.sh` cache/branch-predictor defaults (D/I cache 512 lines, assoc 1, L2 0, bp 16). Record settings with every result.
- why: Baseline for later optimization work. Tuning is out of scope.
- alternatives:
  - Sweep cache sizes: optimization work, later activity.

## Halt via cortos2 `ta 0`, no timeout
- level: major-implementation-detail
- chosen: cortos2 `init_footer.s.tpl` issues `ta 0` after init calls return; C-model halts and prints stats. ResNet-50 run has no timeout; run detached with a log file.
- why: User wants the run to finish. `ta 0` is the existing end marker.
- alternatives:
  - Timeout + layer sampling: estimate, not measurement.

## Shared C-model metrics parser
- level: design-choice
- chosen: One script `os/rtos/cortos2/examples/tflite/cmodel_metrics.py`: input C-model log; output metric table plus pass check against `expected_uart.txt` lines. Used for all three workloads.
- why: Same format for every result; runnable e2e check.
- alternatives:
  - Manual log reading: not repeatable.
  - Script inside cortos2 package: wider change than needed.

## Docs in three places
- level: design-choice
- chosen: New `docs/cortos2-cmodel-metrics.md`; C-model section in `docs/tflite-micro/resnet50-cortos2.md` (both `repos/ajit-toolchain`); update aparajit `docs/resnet50-metrics.md` with measured figures.
- why: Generic metric guide, model runbook, and estimate-vs-measure each have an existing home.
- alternatives:
  - Toolchain docs only: leaves the paper estimate as headline.

## cortos2 TFLM guide: verified C-model rows only
- level: design-choice
- chosen: `docs/tflite-on-cortos2.md` documents qemu for all 20 examples; C-model verified for `hello_world` and `resnet50`; generic C-model recipe; others marked `not run`.
- why: One entry point for both targets without 18 extra C-model runs. CoRTOS v1 `micro_speech` C-model was a skip.
- alternatives:
  - C-model scripts and runs for all 20: large, slow, outside the metrics goal.
  - Four main models only: still adds unverified-risk runs (`micro_speech`).

## Per-layer sampling instead of a full run
- level: design-choice
- replaces: Halt via cortos2 `ta 0`, no timeout
- chosen: Measure each distinct layer configuration on the C-model, cropped; total = Σ ops × measured cycles per MAC/element. `ta 0` halt still ends each run.
- why: C-model ~68k instr/s (without `-w`); full ResNet-50 ~41e9–62e9 instr → 7–11 days.
- alternatives:
  - Full run detached: days of wall-clock, single data point late.
  - Speed up C-model: source edits out of scope.

## Direct reference-kernel harness
- level: design-choice
- chosen: `resnet50_layers` cortos2 example calls TFLM int8 reference kernels directly with the real op's weights/quant from the embedded `.tflite`; `LAYER=`, `ROWS=` at build time.
- why: Same kernel code ResNet-50 runs (TFLM kernels `-O2` = cortos2 `OptimizationLevel: 2`). No TF/converter in container for single-op `.tflite` generation.
- alternatives:
  - Cropped single-op `.tflite` via MicroInterpreter: needs TF or flatbuffer editing; adds dispatch overhead.

## Crop to output rows, verify linearity
- level: major-implementation-detail
- chosen: `ROWS=1` for all 23 conv configurations; linearity check at 1/2/4 for two configurations (≤ ~5% drift).
- why: Rows are uniform work; 1 row = ~1–8M MACs → 3–25 min per run.
- alternatives:
  - Full layers: hours each.
  - Top-10 configurations only: leaves ~20% of MACs extrapolated by analogy.

## qemu libinsn instruction anchor
- level: major-implementation-detail
- chosen: qemu-ajit `libinsn.so` plugin counts full-run and skip-invoke instructions; invoke instructions × weighted CPI cross-checks the per-layer sum (15% gate).
- why: Exact whole-model instruction count is cheap (~90 s qemu); independent check on sampling.
- alternatives:
  - C-model sampling alone: no independent total.

## Process write-up in `docs/resnet50-metrics.md`
- level: design-choice
- chosen: aparajit `docs/resnet50-metrics.md` carries the measured estimate plus the full process and logic; toolchain docs carry how-to.
- why: User asked for process and logic in the ResNet-50 metrics doc.
- alternatives:
  - Process only in toolchain runbook: splits estimate from its derivation.

## Reuse the cortos2 ResNet-50 example
- level: design-choice
- chosen: Add `build.sh` / `run.sh` beside `build_qemu.sh` / `run_qemu.sh` in `os/rtos/cortos2/examples/tflite/resnet50/`. Same app, model, `INPUT=` embed, `gen/`.
- why: One app for both targets; `hello_world` already has this layout.
- alternatives:
  - Separate example dir: duplicates the model and app.

## Layer harness dir; ResNet-50 example gets only a skip-invoke knob
- level: design-choice
- replaces: Reuse the cortos2 ResNet-50 example
- chosen: New `os/rtos/cortos2/examples/tflite/resnet50_layers/` links the ResNet-50 example's `.tflite` blob. ResNet-50 example gains opt-in `SKIP_INVOKE=1` for `build_qemu.sh` (qemu instruction anchor); no C-model scripts there.
- why: Full-model C-model run dropped. Harness is a different app; sharing the blob avoids a second model copy.
- alternatives:
  - Layer mode inside `resnet50_main.cc`: mixes two apps and two build flows.

## KernelRunner over TFLM registrations
- level: design-choice
- replaces: Direct reference-kernel harness
- chosen: Harness runs each op via TFLM `KernelRunner` + `Register_<OP>()` (Init/Prepare/Invoke) linked from `libtensorflow-microlite.a` (`KernelRunner` symbols present). `VARIANT` / `TFLM_LIB_DIR` select the library.
- why: Future optimized kernels land in the TFLM build (`OPTIMIZED_KERNEL_DIR`); harness measures them unchanged. Prepare computes per-channel quant itself.
- alternatives:
  - Direct reference header calls: never sees optimized kernels; duplicates Prepare logic.

## Golden checksums gate variants
- level: major-implementation-detail
- chosen: Reference sweep writes per-layer output checksums; variant results accepted only on exact match.
- why: int8 TFLM kernels must be bit-exact across implementations; speed without correctness is meaningless.
- alternatives:
  - Tolerance compare: hides real bugs in int8 paths.

## Analytic vector speedup, last step
- level: design-choice
- chosen: `vector_estimate.py` + assumptions file applied to measured reference per-config costs; conservative / expected / optimistic range. Last step before e2e.
- why: ISA v2 SIMD not in C-model or FPGA prototype; vector kernels months away. Template lets the estimate be rerun as assumptions firm up.
- alternatives:
  - Wait for hardware: no early guidance.
  - Hand-written single number: not reproducible.

## LayerRunner and host-side layer generation
- level: major-implementation-detail
- replaces: KernelRunner over TFLM registrations
- chosen: Same idea (TFLM `Register_<OP>()` Init/Prepare/Invoke from `libtensorflow-microlite.a`, `VARIANT` / `TFLM_LIB_DIR` pick the library), but through a small local `LayerRunner` (TFLM `FakeMicroContext` + `MockMicroGraph`, 256 KB arena). Op parameters are extracted on the host by `generate_layer.py` into a header + cropped constant blob instead of parsing the model in the app.
- why: `KernelRunner`'s 10000 B arena fails Prepare on 2048-channel convs; host extraction keeps the target app small and the blob limited to one op's cropped weights.
- alternatives:
  - Patch `KernelRunner` arena size: edits TFLM without need.
  - In-app `tflite::GetModel`: links the full 25 MB model into every layer build.

## qemu anchor via stoptrigger at the Invoke call
- level: design-choice
- replaces: Layer harness dir; ResNet-50 example gets only a skip-invoke knob
- chosen: Harness stays in `resnet50_layers/`. The anchor runs the unchanged ResNet-50 qemu ELF twice with `libinsn.so` + `libstoptrigger.so,addr=<pc>`: once at the `call MicroInterpreter::Invoke` in `main`, once at its return address (call + 8). Invoke instructions = difference. Addresses are read from the ELF with `sparc-linux-objdump`.
- why: `libinsn` alone counts until qemu is killed (post-program spin; hello_world 192M vs 1.5M real). Exact PC-bounded counts need no `SKIP_INVOKE` define and no edit to the ResNet-50 example.
- alternatives:
  - `SKIP_INVOKE=1` build + kill-time counts: non-deterministic tail; edits the example.
