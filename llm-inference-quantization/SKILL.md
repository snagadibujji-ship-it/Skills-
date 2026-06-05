---
name: llm-inference-quantization
description: Expert guidance for LLM optimization, including custom Triton/CUDA kernels, advanced quantization techniques (AWQ, GPTQ, EXL2), and VRAM bandwidth management. Triggers on "quantization", "triton kernel", "vram optimization", "awq", "gptq", "fp8", "bitmasking".
---

# LLM Inference & Quantization

## Purpose
This skill provides advanced guidance for AI engineers focused on squeezing maximum performance and efficiency out of Large Language Models. It covers low-level GPU optimization, custom kernels, and state-of-the-art quantization strategies.

## When to Use
- Optimizing model serving for high throughput or low latency.
- Fitting large models onto consumer-grade hardware (VRAM optimization).
- Writing custom Triton or CUDA kernels for non-standard layers.
- Investigating accuracy drops after quantization.

## Key Information

### 1. Quantization Paradigms
*   **Weight-Only Quantization (GPTQ, AWQ)**: Only model weights are quantized. AWQ (Activation-aware Weight Quantization) is often preferred for maintaining accuracy without retraining.
*   **Weight-Activation Quantization (SmoothQuant)**: Both weights and activations are quantized (e.g., INT8/W8A8). Critical for compute-bound scenarios.
*   **KV Cache Quantization**: Quantizing the Key-Value cache (usually to INT8 or FP8) to save memory during long-context inference.

### 2. Formats & Standards
*   **FP8 (E4M3, E5M2)**: The modern standard for H100+ hardware, offering near-FP16 accuracy with INT8 speeds.
*   **GGUF**: The standard for CPU/Metal/Vulkan inference via `llama.cpp`. Supports mixed-precision k-quants.
*   **EXL2**: High-performance format for AutoGPTQ/ExLlamaV2, specialized for fine-grained bitrates (e.g., 4.65 bpw).

### 3. Triton Kernels
Triton allows writing high-performance GPU kernels in Python-like syntax.
*   **Memory Coalescing**: Ensure threads in a warp access adjacent memory locations.
*   **Shared Memory**: Use `tl.load` and `tl.store` to minimize global memory access.
*   **Autotuning**: Use `@tron.autotune` to find the best block sizes and num_warps for specific GPU architectures.

### 4. VRAM Bandwidth Management
LLM inference is almost always memory-bandwidth bound, not compute-bound.
*   **Operator Fusion**: Combining multiple layers (e.g., Linear + Activation) into a single kernel to reduce global memory I/O.
*   **Speculative Decoding**: Using a small "draft" model to predict tokens, which are then verified in parallel by the large model to bypass the memory bottleneck.

### 5. Essential Tools
*   **`AutoAWQ` / `AutoGPTQ`**: Standard libraries for weight-only quantization.
*   **`Triton`**: For writing custom GPU operators.
*   **`NVIDIA Nsight Systems`**: Profile GPU execution to identify stalls and bank conflicts.

## Best Practices
- ✅ **Profile First**: Always identify if your bottleneck is Compute, Memory Bandwidth, or VRAM capacity before optimizing.
- ✅ **Calibration Data**: Use a representative "calibration dataset" (e.g., Pile, C4) during quantization to minimize perplexity loss.
- ✅ **FP8 for H100s**: If using Hopper architecture, prioritize FP8 over INT8 for significant speedups with easier implementation.
- ❌ **Avoid Naive INT8**: Simple rounding (RTN) usually destroys model performance; always use AWQ or GPTQ which adjust weights to compensate for quantization errors.
