# Design Decisions

## Separate tflite board config
- level: design-choice
- chosen: `ajit1-qemu:tflite` with `CONFIG_RAM_SIZE=134217728` and its own linker script: `ram` origin `0x00100000`, length 128M. `nsh` and `smp` keep the 16M `linksparc.ld`.
- why: QEMU already passes `-m 128M`, but `nsh` only maps 16 MiB. ResNet-50 needs the ~25 MiB blob plus a 64 MiB arena. Widening the shared script would change the `nsh` link that `nuttx-ajit-qemu` tests.
- alternatives:
  - Raise `nsh` to 128 MiB: changes the image that activity already checks.
  - Boot ResNet with `-m 256`: cortos2 already fit in 128 MiB.

## NSH argument is an input file
- level: design-choice
- chosen: Four executables under `/tflite/`. Models stay in the ELF. Inputs are files on a ROMFS mounted at `/tflite`, passed as a path. Host folder `tflite-inputs/` is copied onto `/tflite/inputs/` at build; the same relative path replaces a shipped file. `README.md` in that directory says how to invoke each one. `help` prints usage and does not run. `ls /tflite` lists the four names.
- why: The session must accept an input on the NuttX terminal without a rebuild, and the input must be a file the program reads rather than a name compiled into the executable.
- alternatives:
  - Link every sample into the ELF and select it by a short name: no file to feed.
  - cortos2 `INPUT=` rebuild: one JPEG per ELF, no prompt.
  - Bare builtin names (`hello_world 0`): not a directory on the image.

## Blobs stay in the toolchain tree
- level: major-implementation-detail
- chosen: The `tflite` build reads the microlite archive, the existing hello / speech / person model genfiles, and the cortos2 ResNet blob, JPEGs, preprocess spec, and manifest. One script objcopies the model and writes labels. Another writes input files into a ROMFS (uint8 224×224 for ResNet, `label_offset` 1). Generated outputs stay untracked.
- why: Those bytes already exist. A second committed copy adds a 25 MiB blob to the `nuttx` fork.
- alternatives:
  - Commit the `.tflite` into `nuttx`: duplicates the cortos2 file.
  - Fork Apache `apps` to hold the data: no bb-dev-blocks apps repo.

## Programs live in the nuttx fork
- level: design-choice
- chosen: Defconfig, tflite linker script, and the four programs live under `repos/ajit-toolchain/nuttx` on `ajit_main`. Doc and `scripts/run-nuttx-ajit.sh` / `scripts/test-nuttx-ajit.sh` modes live in `repos/ajit-toolchain`. Apache `apps` is not edited.
- why: `apps` is an upstream pin. `nuttx-ajit-qemu` already commits AJIT board code on the bb-dev-blocks `nuttx` fork.
- alternatives:
  - New nuttx-apps fork: extra remote this project has avoided.
  - Drop sources only in `ajit-toolchain` outside `nuttx`: NuttX would not build them as builtins.

## Int8 sine points for hello_world
- level: design-choice
- chosen: Int8 hello-world model only. Inputs `0`, `1`, `2` are x = `0`, `1.57`, `3.14`. Epsilon `0.05`, same bound as `hello_world_test.cc`.
- why: Those three points are the start, middle, and end of a sine half-cycle. The float model’s golden set (`0`, `1`, `3`, `5`) does not match that command list.
- alternatives:
  - Run both float and int8 models: extra results the terminal contract does not name.
  - Use the int8 golden set `0.77`, `1.57`, `2.3`, `3.14` as four inputs: more than the agreed three names.

## Pass lines on NSH stdout
- level: major-implementation-detail
- chosen: The four commands print the pass lines with NSH stdio. Link the existing microlite archive. Add sized `operator delete`, and run `.ctors` / `.init_array`, when that image does not already.
- why: The script matches those lines. The archive’s `DebugLog` already hits UART `0xFFFF3204`, the same console, but it is not the pass contract. cortos2 hung when constructor pointers were unaligned.
- alternatives:
  - Reuse the cortos2 `*_test.cc` mains: they print `~~~ALL TESTS PASSED~~~` and do not take an NSH argument.
