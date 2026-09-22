# Requirement Definition

## Existing Docker image
- kind: internal-behavior
- Build and run use the existing `ajit_build_dev:1.0` image from `ajit-toolchain-setup` (Ubuntu 24.04, Buildroot 2025.02.18, `sparc-linux-gcc` 13.4.0).
- Do not rebuild `ajit_base` or `ajit_build_dev` for this activity.

## Toolchain submodules
- kind: software-interface
- `repos/ajit-toolchain/nuttx` is a submodule whose `origin` is `git@github.com:bb-dev-blocks/nuttx.git` (`master`).
- `repos/ajit-toolchain/apps` is a sibling submodule of Apache nuttx-apps so the kernel's default `../apps` path resolves.
- `ajit_build_dev` sees both because it bind-mounts the toolchain repo. This activity does not build via `aparajit-docker`.

## Live history has no Anshuman
- kind: internal-behavior
- On the live `nuttx` submodule, no commit author or committer contains `Anshuman`.
- New commits use author and committer `bb-dev-blocks <basicblocksdevelopers@gmail.com>`.
- Upstream NuttX authors on the fork stay as published. This activity does not rewrite them.

## Single-core NSH
- kind: end-user-interface
- `ajit1-qemu:nsh` links at `0x00100000` and uses qemu UART0 at `0xFFFF3200` (`tx +0x04`, `rx +0x08`).
- `-smp 1` with `-M ajit1_generic -cpu AJIT1 -serial stdio -nographic` reaches `nsh>` and answers `help` and `hello`.

## Multi-core bring-up
- kind: end-user-interface
- `ajit1-qemu:smp` at `-smp 2` shows CPU1 online on the UART.
- The same image at `-smp 4` shows CPU1, CPU2, and CPU3 online.
- No required run above 4 CPUs.

## Manual NSH terminal
- kind: end-user-interface
- A host command starts qemu in the foreground with the terminal attached to `nsh>`.
- `ajit1-qemu:nsh` (`-smp 1`): type `help` and `hello`.
- `ajit1-qemu:smp` (`-smp 2`): type `ps`.
- Quit with Ctrl-A, then X.

## LEON configs still cross-compile
- kind: internal-behavior
- `s698pm-dkit:nsh`, `s698pm-dkit:smp`, and `xx3823:nsh` still produce a 32-bit SPARC ELF in `ajit_build_dev`.
- Those images do not have to boot on qemu-ajit.

## Process doc
- kind: end-user-interface
- `docs/nuttx-on-ajit-qemu.md` records checkout, author identity, the Docker build, scripted runs, and the manual terminal.
- `repos/nuttx` is not deleted here. The doc says that tree is unused and can be removed later.
