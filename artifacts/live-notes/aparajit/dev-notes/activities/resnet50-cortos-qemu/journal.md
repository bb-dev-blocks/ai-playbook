# Journal

## Pause (Active → Paused)

- why: user pause-work after Goal e2e; product still uncommitted
- done:
  - Qualcomm TFLITE w8a8 ResNet-50 on CoRTOS qemu-ajit; hopper `invoke: ok` + `top1: 457 bow tie`
  - fetch pin https://huggingface.co/qualcomm/ResNet50; host precheck five JPEGs; `docs/tflite-micro/`
  - SPARC unaligned FlatBuffers `memcpy` in TFLM `flatbuffers.patch`; `example_001` still Hello There
- next: commit the three trees, then `mark-completed`
- watch: three repos (`ajit-toolchain`, `tflite-micro`, aparajit); keep `gen/` and the Hub zip out of git
