# machine/ajit1 — Dev-Guide

Canonical **AJIT-1 SoC** description for aparajit: memory map, peripherals, IRQs, and QEMU parameters. Intended for NuttX stage-2, bare-metal firmware, and cross-checking CoRTOS / TFLM bring-up.

## Notes

- **Simulation source of truth**: `qemu-ajit-v1.0` machine `ajit1_generic` (`hw/sparc/ajit1.c`).
- **32-bit SPARC V8**, big-endian. CPU type `AJIT1`.
- **RAM for `-kernel` loads** starts at `0x00100000` (1 MiB PROM at `0x0`). Link bare-metal / NuttX images accordingly.
- **UART layout**: QEMU uses compact registers (`ctrl +0x00`, `tx +0x04`, `rx +0x08`). Legacy CoRTOS header `cortos/res/references/Ajit_Device_Addresses.h` uses `+0x10` / `+0x20` — do not mix layouts on the same target.
- **NuttX port**: not started. Use `profiles.nuttx_qemu` in YAML and copy constants from `include/ajit1_machine.h` into `arch/sparc/src/ajit1/` when adding the chip.

## Artifacts

| Name | Description |
|------|-------------|
| `ajit1-machine.yaml` | Machine definition (memory, peripherals, profiles, NuttX hints) |
| `include/ajit1_machine.h` | C header for firmware and future NuttX chip headers |
| `../qemu-ajit-v1.0/` | QEMU fork implementing this map |
| `../../nuttx-ajit.md` | NuttX integration handoff |
| `../../cortos/` | CoRTOS (co-operative RTOS); different default link bases |

## Quick reference

```bash
# Run a bare-metal ELF (after building qemu-ajit)
qemu-ajit-v1.0/build/qemu-system-sparc \
  -M ajit1_generic -cpu AJIT1 -serial stdio -nographic -m 256M \
  -kernel path/to/firmware.elf
```

| Device | Base | IRQ |
|--------|------|-----|
| INTC | `0xFFFF3000` | — |
| Timer | `0xFFFF3100` | 10 |
| UART0 | `0xFFFF3200` | 12 |
| Scratchpad | `0xFFFF2C00` | — |
