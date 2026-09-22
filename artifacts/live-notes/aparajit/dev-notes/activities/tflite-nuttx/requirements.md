# Requirement Definition

## NSH model commands
- kind: end-user-interface
- One image: `ajit1-qemu:tflite`, `-smp 1`, `nsh>` prompt.
- Programs are executables in `/tflite/` on that image: `/tflite/hello_world`, `/tflite/micro_speech`, `/tflite/person_detection`, `/tflite/resnet50`.
- `ls /tflite` lists those four names.
- Each program takes a file path. `help`, `-h`, `--help`, or no argument prints usage and does not run.
- `hello_world` files: `/tflite/inputs/hello_world/0`, `1`, `2` (text `0`, `1.57`, `3.14`).
- `micro_speech` files: `/tflite/inputs/micro_speech/yes`, `no` (1000 ms, big-endian int16).
- `person_detection` files: `/tflite/inputs/person_detection/person`, `no_person` (9216 raw bytes).
- `resnet50` files: `/tflite/inputs/resnet50/hopper`, `tench`, `spaniel`, `tabby`, `panda` (150528 raw bytes).
- `/tflite/README.md` tells how to invoke each program. `cat /tflite/README.md` shows it.
- Host folder `tflite-inputs/` is copied to `/tflite/inputs/` at image build. The same relative path replaces a shipped sample. `docs/tflite-on-nuttx.md` states the four file formats, including the ResNet-50 224×224 RGB layout. The `/tflite` volume stays within 8 MiB.
- A missing file: one usage line, no inference.
- Manual session: `./scripts/run-nuttx-ajit.sh tflite`. Quit: Ctrl-A then X.

## Terminal pass lines
- kind: end-user-interface
- Invoke as `/tflite/<name> <file>`.
- `/tflite/hello_world`: int8 sine. The file holds x. The shipped files are x = `0`, `1.57`, `3.14`. Line `hello_world: x=<x> y=<y>`. Pass when `abs(y - sin(x)) <= 0.05`.
- `/tflite/micro_speech` on the yes file prints `micro_speech: yes`. The no file prints `micro_speech: no`.
- `/tflite/person_detection` on the person file prints `person_detection: person`. The no_person file prints `person_detection: no person`. Choice is the higher score.
- `/tflite/resnet50` prints `top1: <id> <label>` from the cortos2 manifest: hopper `457 bow tie`, tench `0 tench`, spaniel `217 English springer`, tabby `285 Egyptian cat`, panda `388 giant panda`.
- ResNet script cap: 700s per invoke.

## How-to document
- kind: end-user-interface
- Path: `repos/ajit-toolchain/docs/tflite-on-nuttx.md`.
- States how the four programs were brought onto single-core NSH.
- States how to test: manual commands and the scripted modes.

## tflite config leaves nsh and smp
- kind: software-interface
- New config `ajit1-qemu:tflite`. `CONFIG_RAM_SIZE=134217728`. Link origin `0x00100000`. Region length 128 MiB for this config only.
- `ajit1-qemu:nsh` stays `CONFIG_RAM_SIZE=16777216`. `boards/sparc/ajit1/ajit1-qemu/scripts/linksparc.ld` stays 16M.
- `ajit1-qemu:smp` unchanged.
- `./scripts/test-nuttx-ajit.sh nsh` still shows `help` and `hello`.

## Toolchain blobs, no second copy
- kind: software-interface
- Link `tflite-micro/gen/ajit_sparc_default_gcc/lib/libtensorflow-microlite.a`.
- Hello, speech, and person model data come from that tree's existing genfiles.
- ResNet sources: `os/rtos/cortos2/examples/tflite/resnet50/resnet50_int8.tflite`, the five JPEGs, `inputs/preprocess_spec.json`, `inputs/manifest.tsv`.
- Do not commit a second copy of those blobs into `nuttx`. Generated arrays stay untracked.
- Missing source path: build exits non-zero and prints that path.
- Do not edit cortos2 example sources. Edit `tflite-micro` only when one of these four programs cannot link or fails on a SPARC bug.

## One image holds every input file
- kind: internal-behavior
- Models stay in the one `tflite` ELF. Inputs are files on the `/tflite` ROMFS. Switching input does not rebuild.
- One command runs at a time. ResNet arena is 64 MiB.
- Pass lines go to NSH stdout.
- Programs and the `tflite` defconfig live in `repos/ajit-toolchain/nuttx`. The booted image exposes the programs only under `/tflite/`, not as bare `nsh>` names.
- Doc and script modes live in `repos/ajit-toolchain`. No commits under `apps/`.
