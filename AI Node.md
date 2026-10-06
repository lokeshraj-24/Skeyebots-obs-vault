
## Jetson Orin AGX

Has 2048 core in GPU,
Additionally 2 DLA (Deep learnign accelerator)

DLA:
- Built especially for deep learnign tasks liek convolution, pooling and such
- way more power efficient
- precision: INT8 (best), FP16
- best for optimized models like YOLO
- might not run complex models involving special layers

Ides:
- make a complex model for EO, run it in GPU
- Two highly optimized, simple model for IR/SWIR, and offload it in DLA