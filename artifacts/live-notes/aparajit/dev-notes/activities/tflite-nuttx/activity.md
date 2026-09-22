# TFLite on single-core NuttX

| Key | Value |
|---|---|
| status | Active |
| slug | tflite-nuttx |
| branch | none |
| ticket | none |
| notes | commits only when asked at the end |

# Goal

`ajit1-qemu:tflite` boots single-core NSH on qemu-ajit. The image has `/tflite/hello_world`, `/tflite/micro_speech`, `/tflite/person_detection`, and `/tflite/resnet50`. Each executable reads an input file and prints a stable result. `/tflite/README.md` says how to invoke them. `repos/ajit-toolchain/docs/tflite-on-nuttx.md` explains how that port works and how to test it. `ajit1-qemu:nsh` and `ajit1-qemu:smp` stay as they are.

# Scope

NuttX step after `nuttx-ajit-qemu`, using the cortos2 TensorFlow Lite and ResNet-50 results as the behavior source. In: one new single-core config at 128 MiB, four NSH programs, every agreed input as a file on that image, build reads the existing toolchain blobs, `run` / `test` script modes, and the how-to doc above.

Out: the sixteen kernel unit tests, SMP, the C-model, FPGA, training, new SPARC kernels, edits to cortos2 examples, commits into Apache `apps`, a second committed copy of the model blobs, and any change to `nsh` or `smp`. Commits wait until asked at the end.

# Background and Special Notes

- `nuttx-ajit-qemu` is Active. Its scope lists TensorFlow Lite as out. QEMU already passes `-m 128M`. `ajit1-qemu:nsh` sets `CONFIG_RAM_SIZE=16777216`. `boards/sparc/ajit1/ajit1-qemu/scripts/linksparc.ld` gives `ram` origin `0x00100000` and length 16M. That region is shared by `nsh` and `smp`.
- `nuttx` is `repos/ajit-toolchain/nuttx`, branch `ajit_main`, origin `git@github-bb:bb-dev-blocks/nuttx.git`. `apps` is the Apache pin `bc0ed23a5dea42e9aa42dfccc372fe9f4654d10d` (detached). No bb-dev-blocks nuttx-apps repo.
- Microlite archive: `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`, built `TARGET=ajit` with `sparc-linux-`. `DebugLog` writes UART TX `0xFFFF3204`, the same UART as NSH. cortos2 needed `.ctors` aligned to 4 bytes and a sized `operator delete` when libstdc++ is not linked.
- ResNet-50 assets stay in `os/rtos/cortos2/examples/tflite/resnet50/`: `resnet50_int8.tflite` (~25 MiB), five JPEGs, `inputs/preprocess_spec.json` (uint8 224×224, `label_offset` 1), `inputs/manifest.tsv`. cortos2 measured invoke on the order of 90s and used a 64 MiB arena inside 128 MiB. `resnet50-cortos2-qemu` is still Active; this activity reads those files and does not edit that example.
- `.dev-notes/definition.md` is `TODO`. Fit is the activity chain, confirmed for this slug.

# Current Design

- New board config `ajit1-qemu:tflite` only. `CONFIG_RAM_SIZE=134217728`. Its linker region is `ram` at `0x00100000`, length 128M. Do not change `linksparc.ld` or the `nsh` / `smp` defconfigs.
- Four programs in the `nuttx` tree (not under `apps/`), installed on the image as `/tflite/hello_world`, `/tflite/micro_speech`, `/tflite/person_detection`, and `/tflite/resnet50`. Models stay in the ELF. Inputs are files under `/tflite/inputs/`. A host folder `tflite-inputs/` is copied there at build time; the same relative path replaces a shipped sample. Invoke `/tflite/<name> <file>`. `cat /tflite/README.md` is the on-image how-to. `ls /tflite` lists those four. One command runs at a time. Pass lines use NSH stdout.
- `hello_world` uses the int8 sine model. The shipped files hold x = `0`, `1.57`, `3.14`. Pass: `hello_world: x=<x> y=<y>` and `abs(y - sin(x)) <= 0.05`.
- `micro_speech` on the yes and no files prints `micro_speech: yes` or `micro_speech: no` (1000 ms clips). `person_detection` on the person and no_person files prints `person_detection: person` or `person_detection: no person` from the higher score.
- `resnet50` prints `top1: <id> <label>` from `inputs/manifest.tsv`: hopper `457 bow tie`, tench `0 tench`, spaniel `217 English springer`, tabby `285 Egyptian cat`, panda `388 giant panda`. Arena 64 MiB. Preprocess follows `preprocess_spec.json`. Per-invoke script cap 700s.
- `help` or a missing file: one usage line, no inference.
- Build reads the microlite archive, the existing hello / speech / person genfiles, and the cortos2 ResNet files. Generated arrays stay untracked. Missing path: non-zero exit naming that path.
- Scripts: `./scripts/run-nuttx-ajit.sh tflite` (`-smp 1`, `-m 128M`). Test modes `tflite-boot`, `tflite-small`, `tflite-resnet`, and e2e `tflite`.
- Doc: `repos/ajit-toolchain/docs/tflite-on-nuttx.md` (how the port works, and how to test it).

# Current Plan

1. Add `ajit1-qemu:tflite` and a 128 MiB linker script used only by that config. Boot to `nsh>` at `-smp 1`. Confirm `nsh` still answers `help` and `hello`, and `linksparc.ld` is still 16M.
2. Add the four programs under `/tflite/` on the image, and a toolchain-side generator that objcopies the ResNet blob and embeds all five preprocessed JPEGs. Link microlite and the existing small-model genfiles. Supply sized `operator delete` and run `.ctors` / `.init_array` when the image does not already.
3. `/tflite/hello_world`, `/tflite/micro_speech`, and `/tflite/person_detection`: pass lines for every listed input; usage line for an unknown name.
4. `/tflite/resnet50`: five `top1` lines, 64 MiB arena, 700s cap, image still boots in the 128 MiB guest.
5. Write `docs/tflite-on-nuttx.md`. E2e runs every listed input, then the existing `nsh` check.

# Milestones

1. [x] `ajit1-qemu:tflite` boots NSH at 128 MiB; `nsh` and the 16M link stay
   - tests:
     - tflite defconfig `CONFIG_RAM_SIZE=134217728`; link region length 128M; `-smp 1` reaches `nsh>`
     - `nsh` defconfig still `16777216`; `linksparc.ld` ram length still 16M
     - `./scripts/test-nuttx-ajit.sh nsh` still shows `help` and `hello`
   - evidence:
     - inside `ajit_build_dev`: `./scripts/test-nuttx-ajit.sh tflite-boot`
     - `./scripts/test-nuttx-ajit.sh nsh`

2. [x] Small-model commands print pass lines
   - tests:
     - `ls /tflite` lists `hello_world`, `micro_speech`, `person_detection`, `resnet50`
     - `cat /tflite/README.md` shows how to pass an input file
     - `/tflite/hello_world` on `/tflite/inputs/hello_world/0|1|2` prints `x=0`, `x=1.57`, `x=3.14` and `y` within 0.05 of `sin(x)`
     - `/tflite/micro_speech` on the yes and no files prints those labels
     - `/tflite/person_detection` on the person and no_person files prints `person` and `no person`
     - `help` or a missing file on each of the three programs prints usage and no result line
   - evidence:
     - `./scripts/test-nuttx-ajit.sh tflite-small`

3. [x] ResNet-50 prints manifest top-1 for all five inputs
   - tests:
     - `/tflite/resnet50` on each file under `/tflite/inputs/resnet50/` prints `top1:` with the manifest id and label
     - `/tflite/resnet50 nope` prints usage and no `top1`
     - missing `resnet50_int8.tflite` fails the build and names that path
     - invoke cap 700s; guest stays `-m 128M`
   - evidence:
     - `./scripts/test-nuttx-ajit.sh tflite-resnet`

4. [x] Goal e2e: every listed input, the how-to doc, and `nsh` still passes
   - tests:
     - e2e: `tflite-boot` + every input from milestones 2 and 3 + `docs/tflite-on-nuttx.md` states how the port works and lists the test commands + `nsh` `help` / `hello`
   - evidence:
     - `./scripts/test-nuttx-ajit.sh tflite`
     - from `repos/ajit-toolchain`: `test -f docs/tflite-on-nuttx.md`

# Next Steps

1. File-input checks are committed. Toolchain `marshal_updates` `9a3522f95`, NuttX `ajit_main` `e675caa6ab`, TFLite Micro `sparc-ajit` `dcfbc783`.

# References

- `.dev-notes/activities/nuttx-ajit-qemu/activity.md` — context-only. Learned: single-core NSH is `ajit1-qemu:nsh` at `-smp 1` via `./scripts/run-nuttx-ajit.sh nsh` and `./scripts/test-nuttx-ajit.sh nsh`. TensorFlow Lite was left out. Board link origin is `0x00100000`; UART0 is `0xFFFF3200`. Build is `CROSSDEV=sparc-linux-` in `ajit_build_dev`.
- `.dev-notes/activities/tflite-cortos2-qemu/activity.md` — context-only. Learned: reuse `libtensorflow-microlite.a` on `sparc-ajit`. SPARC g++ `.ctors` must be 4-byte aligned, and sized `operator delete` is required without libstdc++. Selected runnable models here are `hello_world`, `micro_speech`, and `person_detection`. Kernel tests stay out.
- `.dev-notes/activities/resnet50-cortos2-qemu/activity.md` — context-only. Learned: int8 ResNet-50 v1 uses the cortos2 `resnet50_int8.tflite`, five JPEGs, a 64 MiB arena, and 128 MiB guest RAM. Pass is stable top-1 id and label. Script cap 700s. Do not edit that example tree.
