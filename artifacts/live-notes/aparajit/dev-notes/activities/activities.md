# Activities

High-level catalog. Details live in each activity folder.

## ajit-toolchain-setup: AJIT toolchain Docker on Mac (arm64)

Done state: native arm64 Docker (`ajit_base` / `ajit_build_dev`) on Ubuntu 24.04, Buildroot 2025.02.18, five CoRTOS examples on the C simulator. Aparajit root docker is out of scope.

## cortos-qemu: CoRTOS examples on qemu-ajit in the toolchain

Vendor qemu-ajit into ajit-toolchain, build it in ajit_build_dev, and run CoRTOS examples on QEMU or the C simulator via separate scripts. Incremental coverage starting at example_001; skips documented.

## tflite-cortos-qemu: TFLite Micro on CoRTOS and qemu-ajit

Pin TFLM as an ajit-toolchain submodule on bb-dev-blocks sparc-ajit, add TARGET=ajit, and run selected tests on qemu-ajit then CoRTOS (C-model only for small cases).

## resnet50-cortos-qemu: ResNet-50 on CoRTOS and qemu-ajit via TFLM

Run int8 ImageNet ResNet-50 through TFLite Micro on CoRTOS and qemu-ajit, with a dedicated example folder, five build-time-selectable inputs, and recipe docs for other TFLM models. Saved blob: Qualcomm AI Hub TFLITE w8a8.

## cortos2-qemu: CoRTOS2 examples on qemu-ajit in the toolchain

Port cortos2 examples to qemu-ajit inside ajit_build_dev: --target qemu, dual C-model/qemu scripts, all eleven examples green on both.

## tflite-cortos2-qemu: TFLite Micro on cortos2 qemu-ajit

Run the selected TFLite Micro tests on cortos2 and qemu-ajit, reusing the shipped microlite archive, and add the seven kernel tests the ResNet-50 resolver uses that the existing nine do not cover.

## resnet50-cortos2-qemu: ResNet-50 on cortos2 and qemu-ajit via TFLM

Run the int8 ImageNet ResNet-50 v1 TFLite graph on cortos2 and qemu-ajit, reusing the Qualcomm w8a8 blob and the sparc-ajit microlite library. Build-time JPEG select, 700s qemu cap, guest RAM 128 MiB unless the image does not fit. Runbook: `docs/tflite-micro/resnet50-cortos2.md`.

## nuttx-ajit-qemu: NuttX on ajit-qemu

Build and run NuttX on qemu-ajit from nuttx and apps submodules inside repos/ajit-toolchain, using ajit_build_dev. Single-core NSH first, then SMP at 2 and 4 CPUs, plus a manual terminal and a process doc. repos/nuttx stays until a later removal.

## tflite-nuttx: TFLite on single-core NuttX

Run hello_world, micro_speech, person_detection, and int8 ResNet-50 from `/tflite/` on single-core NuttX (`ajit1-qemu:tflite`). An NSH path picks one stored input and prints the result. How-to: `repos/ajit-toolchain/docs/tflite-on-nuttx.md`. `nsh` and `smp` stay unchanged.

## drone-pitch-deck: Drone pitch deck 2026 (Ajit + Apara)

Refine agent-drafted drone silicon pitch docs into an investor package for a seed round: verified references and market study, two-scenario cost model, concrete MVPs, investor brief and Q&A (India first, Europe later), a professional Beamer/TikZ deck and speaker notes. Replanned: single 28 nm Ajit + Apara SoC at 800 MHz, MobileNetV3 focus, Agam Robotics and ideaForge partners, leaner costs (~₹13–20 Cr).

## resnet50-cortos2-cmodel: ResNet-50 on cortos2 and the Ajit C-model: cycles and metrics

Fix a C-model metric set from cortos2 example_001 and tflite hello_world, then estimate C-model cycles for one int8 ResNet-50 inference by measuring each distinct layer configuration (resnet50_layers harness, cropped rows), summing by topology, and cross-checking with a qemu libinsn instruction count; process documented in docs/resnet50-metrics.md. Harness reusable for future kernel variants (KernelRunner, golden checksums, compare.py); last step estimates ISA v2 vector-instruction speedup analytically. Adds docs/tflite-on-cortos2.md, a qemu and C-model run guide for all cortos2 TFLM examples. First measured CNN performance baseline on Ajit; optimizations out of scope.
