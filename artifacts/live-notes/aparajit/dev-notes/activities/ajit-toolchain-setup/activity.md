# AJIT toolchain Docker on Mac (arm64)

| Key | Value |
|---|---|
| status | Complete |
| slug | ajit-toolchain-setup |
| branch | none |
| ticket | none |
| notes | |

# Goal

Native `linux/arm64` Docker (`ajit_base` / `ajit_build_dev`) on **Ubuntu 24.04**, SPARC V8 32-bit uClibc from **Buildroot 2025.02.18**, five CoRTOS examples on the **C simulator**. Product edits only in `repos/ajit-toolchain`. 16.04 / Buildroot 2014.08 were stepping stones.

# Scope

Unchanged: toolchain submodule Docker + Darwin clone/volume + 16.04 then 24.04 + sibling `buildroot_src_2025.02/`. Out: aparajit `docker/`, host Docker install, macOS native setup, `buildroot_src_64`, qemu-ajit/FPGA, TFLite/NuttX, baking `ajit_build`/`ajit_tools`, amd64 on this Mac.

# Background and Special Notes

- Darwin volume `ajit-toolchain-build` at `$AJIT_HOME/build`. No `getent group docker`. Distro `python3` **3.12.3**. Nested AHIR pin `0816fb6d533715d89364551c267642df701b391c`. Do not commit arm64 `ahir_release` overlays or parent `buildroot-2025.02.18.tar.gz`.
- `pt_load_sections.py` matches SHF_ALLOC PROGBITS by vaddr (not pyelftools `section_in_segment`). `compileToSparcUclibc.py` `-fno-pic -fno-pie`.
- `example_250` reader must not `ldstub`-spin `cortos_readMessage` during the writer’s first UART log (C-model `Serial_Tx` starve / `TX_FULL` livelock). UART is stdout and dumps at halt; MMU `Info:` is stderr. No `killall` in the image: `pkill -x ajit_C_system_m`. Do not `pkill -f ajit_C_system_model` from a line that contains that string. C-model `-d` waits for gdb; `-w` worsens UART starve. Fake segfault = SIGTERM handler.
- `ajit_debug_monitor` scons still fails (not required).

# Current Design

- Host-native image arch. No `--platform`. Container uid/gid = host. Darwin: bind-mount clone + volume on `$AJIT_HOME/build`. Linux: bind-mount only.
- `ajit_base` / `ajit_build_dev` `FROM ubuntu:24.04`. PATH uses `build/buildroot-2025.02.18` (`sparc-linux-gcc` 13.4.0, `-dumpmachine` `sparc-buildroot-linux-uclibc`). `buildroot_src/` 2014.08 kept on disk.
- Must not break: CoRTOS `./build.sh` then `./run.sh` on `example_{001,050,100,150,250}` inside `ajit_build_dev` after `source ./set_ajit_home` and `source docker/ajit_build/ajit_env`.
- Touched (shipped): `docker/ajit_base`, `docker/ajit_build_dev`, `buildroot_src_2025.02/`, `AjitPublicResources/tools/scripts/pt_load_sections.py` (+ compileToSparc uclibc PIE/python), CoRTOS shebang/gas/inline/`cortos build` fail, `os/rtos/cortos/examples/example_250/main.c`. Replication: `docs/m1-ajit-ubuntu16-setup-arm64mac.md`, `docs/m3-ajit-ubuntu24-buildroot2025-setup-arm64mac.md` (M3 text still omits 250).

# Current Plan

Shipped. Reopen only via `resume-work`.

# Milestones

1. [x] Host-native images; Darwin volume; 16.04 try
   - tests: `uname -m` aarch64; uid/gid; volume `Foo`/`foo`; 2014.08 `setup.sh`; AHIR pin; CoRTOS `001`/`050`/`100`/`150` on 16.04 (not Goal e2e)
   - evidence: `docker/ajit_base/build.sh`; `docker/ajit_build_dev/build.sh && ./run.sh`; `file` on arm64 `libPipeHandlerDebugPthreads.so`

2. [x] Ubuntu 24.04 base
   - tests: `VERSION_ID=24.04`; distro `python3`; no 3.6 PPA
   - evidence: `grep VERSION_ID /etc/os-release`; `python3 --version`

3. [x] Buildroot 2025.02.18 SPARC32 uClibc
   - tests: vendored tree; gcc 13 V8 32-bit; setup does not open the tarball
   - evidence: `sparc-linux-gcc -dumpmachine` / `-dumpversion`; docs/m3

4. [x] CoRTOS e2e (Goal)
   - tests: `example_{001,050,100,150,250}` `./build.sh` then `./run.sh` exit 0 on 24.04 + gcc 13 C-model
   - evidence: inside `ajit_build_dev`, `source` `ajit_env`, `cd os/rtos/cortos/examples/<ex>` then `./build.sh && ./run.sh`. `001` Hello There; `100` VALUE_IS 20; `150` four send/receive; `250` acquire/send/receive/release (after queueId wait). `200` also ran (not required).

# Next Steps

1. Push `marshal_updates` (`7702c76ae`) and aparajit (`aae3000`) when asked.
2. Fastest check: `pkill -x ajit_C_system_m`; `cd os/rtos/cortos/examples/example_250 && ./build.sh && ./run.sh` (cwd not `cortos_build`).
3. Safest first edit: keep arm64 `ahir_release` uncommitted; do not `rm -rf cortos_build` while a shell’s cwd is that dir.
4. Optional later: M3 doc still says skip 250; amd64 host; bake `ajit_build`/`ajit_tools`; `ajit_debug_monitor`.

# References

- `docs/m1-ajit-ubuntu16-setup-arm64mac.md` — 16.04 arm64 Mac baseline
- `docs/m3-ajit-ubuntu24-buildroot2025-setup-arm64mac.md` — 24.04 + BR 2025 + CoRTOS `001`/`050`/`100`/`150`
- Latest pin: `repos/ajit-toolchain` `7702c76ae` (`example_250` UART); aparajit `aae3000`. mmap helper `b614173c2` / `1803c13`. M3 `752c624c3` / `06c87e3`. M1 `9bd2bb60a` / `b77940f`. AHIR `0816fb6d`.
- `git@github.com:bb-dev-blocks/ahir.git` — nested `$AJIT_HOME/ahir`
