# Design Decisions

## TFLM lives in the toolchain, not aparajit
- level: design-choice
- chosen: Submodule `$AJIT_HOME/tflite-micro` of `ajit-toolchain`, remote `bb-dev-blocks/tflite-micro`, branch `sparc-ajit`.
- why: Darwin `ajit_build_dev` bind-mounts only the toolchain. Product home for CoRTOS/qemu is already that repo.
- alternatives:
  - Aparajit `repos/tflite-micro` plus extra mounts: fights existing container convention.

## Cherry-pick SPARC commit onto fork main
- level: major-implementation-detail
- chosen: `sparc-ajit` is cherry-pick of `dd4f0b4b` onto fork `main` (`ee368b2a`). Exact original SHA is not the branch tip.
- why: Fork `main` had moved; a branch at `dd4f0b4b` would omit later fork commits. Cherry-pick had no conflicts.
- alternatives:
  - Point `sparc-ajit` at `dd4f0b4b` only: diverges from org `main`.

## Static microlite lib plus thin examples
- level: design-choice
- chosen: Build `libtensorflow-microlite.a` once. Example dirs hold scripts, `config.yaml`, `expected_uart.txt`, trampoline if needed. Test sources stay in the submodule.
- why: CoRTOS is C and flat; TFLM is a C++ tree. Flattening would be a CoRTOS rewrite.
- alternatives:
  - Teach CoRTOS to compile the whole TFLM tree: out of scope.

## New TFLM `TARGET=ajit`
- level: design-choice
- chosen: New make target in the fork for AJIT SPARC V8 32-bit, UART `DebugLog`, bare-metal link. CoRTOS consumes the `.a` only.
- why: `sparc_generic` is Linux qemu-user (often sparc64). Flags would drift if copied into every example makefile. ResNet50 will reuse the lib.
- alternatives:
  - Example-only makefiles, no TFLM target: duplicates flags.

## Example parent under CoRTOS examples
- level: design-choice
- chosen: `os/rtos/cortos/examples/tflite/` with one child project per test (kernels as `kernels/<name>/`). Shared crt0/UART under the parent.
- why: One place for tflite apps; each child is a CoRTOS project. Matches “one example folder” plus CoRTOS flat-project rule.
- alternatives:
  - `$AJIT_HOME/examples/tflite/` outside CoRTOS: splits how to run CoRTOS apps.

## UART `DebugLog` not CoRTOS printf
- level: major-implementation-detail
- chosen: TFLM `DebugLog` writes AJIT UART MMIO (same map as qemu-ajit / CoRTOS). Same test body on bare-metal and CoRTOS.
- why: Bare-metal has no CoRTOS printf. One log path keeps the UART contract identical.
- alternatives:
  - CoRTOS printf on step 3 only: two log implementations.

## CoRTOS start symbol is unmangled
- level: implementation-detail
- chosen: `CortosInitCalls` uses `main`, with `extern "C"` trampoline if a test `main` is unsafe (argc/argv, C++ linkage).
- why: CoRTOS names the symbol in YAML; C++ mangling would miss the entry.
- alternatives:
  - Rename TFLM `main` in every test: extra fork churn.

## SPARC assemble without PIC
- level: implementation-detail
- chosen: `-fno-pic -fno-pie` on crt0 and TFLM C++. gcc default PIC emitted GOT relocs that left TBR/stack as 0 after link.
- why: qemu hung with no UART until `_start` used absolute `sethi %hi(sym)`.
- alternatives:
  - Hand-written `%hi/%lo` without gcc: extra churn.

## CoRTOS ExtraCc / ExtraLibDirs
- level: major-implementation-detail
- chosen: yaml lists of extra `.cc`, `-I`, and `.a` dirs. g++ `-S` then existing `-s`. Linker script gained empty ctor arrays.
- why: CoRTOS still C-only in `compileToSparcUclibc.py`. Flattening TFLM is out of scope.
- alternatives:
  - Teach compileToSparc C++: larger script change.

## Walk C++ `.init_array` before `main`
- level: implementation-detail
- chosen: `tflite_call_ctors` from bare-metal crt0; CoRTOS `tflite_cortos_start` for `TEST()` suites. Walk `.init_array` and `.ctors` (SPARC g++ uses `.ctors`).
- why: `micro_test_v2` registers tests in static constructors. Empty `.init_array` walk printed `ALL TESTS PASSED` with 0 tests.
- alternatives:
  - Rewrite tests to explicit `main`: extra fork churn.
