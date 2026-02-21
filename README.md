# Running a LLM on the ESP32
![LLM on ESP32](/ESP32_LLM.jpg)
![LLM Output](/llm_output.gif)

## Summary
I wanted to see if it was possible to run a Large Language Model (LLM) on the ESP32. Surprisingly it is possible, though probably not very useful.

The "Large" Language Model used is actually quite small. It is a 260K parameter [tinyllamas checkpoint](https://huggingface.co/karpathy/tinyllamas/tree/main/stories260K) trained on the [tiny stories](https://huggingface.co/datasets/roneneldan/TinyStories) dataset.

The LLM implementation is done using [llama.2c](https://github.com/karpathy/llama2.c) with extensive optimizations to maximize token generation speed on the ESP32-S3.

## Hardware
LLMs require a great deal of memory. Even this small one still requires 1MB of RAM. I used the [ESP32-S3FH4R2](https://www.mouser.com/ProductDetail/Espressif-Systems/ESP32-S3FH4R2?qs=tlsG%2FOw5FFjPrwkmZSBQNA%3D%3D) because it has 2MB of embedded PSRAM.

## Optimizing Llama2.c for the ESP32

The following optimizations have been applied to maximize token generation speed:

### Dual-Core Processing
1. Utilizing both cores of the ESP32 during math-heavy operations
2. Parallel matrix multiplication split across both cores
3. Parallel attention head computation across cores

### SIMD and ESP-DSP Optimizations
1. Using [ESP-DSP SIMD dot product functions](https://github.com/espressif/esp-dsp/tree/master/modules/dotprod/float) (`dsps_dotprod_f32_aes3`) for all vector dot products
2. SIMD-optimized RMSNorm using ESP-DSP for sum-of-squares computation
3. SIMD-optimized attention score computation

### Fast Math Approximations
1. Fast inverse square root (Quake III algorithm) for RMSNorm and attention scaling
2. Fast exponential approximation (Schraudolph's method) for softmax
3. Fast sigmoid approximation for SwiGLU activation

### Loop Optimizations
1. 4x loop unrolling for all critical loops (softmax, residual connections, weighted sums)
2. `restrict` keyword for non-aliased pointer optimizations
3. Pre-computed scaling factors to avoid repeated calculations

### Memory and Cache Optimizations
1. Functions marked with `IRAM_ATTR` to place critical code in fast IRAM
2. 64-byte data cache lines for better PSRAM access
3. 32KB instruction cache with 8-way associativity
4. 64KB data cache with 8-way associativity

### Compiler Optimizations
1. `-Ofast` optimization level
2. `-ffast-math` for aggressive floating-point optimizations
3. `-funroll-loops` for automatic loop unrolling
4. `-ftree-vectorize` for auto-vectorization
5. `-finline-functions` for aggressive function inlining

### System Configuration
1. CPU speed at 240 MHz
2. PSRAM speed at 80 MHz
3. FreeRTOS tick rate at 1000 Hz for responsive task switching


## Setup
This requires the [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/index.html#installation) toolchain to be installed

```
idf.py build
idf.py -p /dev/{DEVICE_PORT} flash
```


