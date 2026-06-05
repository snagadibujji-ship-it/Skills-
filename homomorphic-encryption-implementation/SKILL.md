---
name: homomorphic-encryption-implementation
description: Expert guidance for Fully Homomorphic Encryption (FHE) to compute on encrypted data using libraries like Zama Concrete or Microsoft SEAL. Triggers on "fhe", "homomorphic encryption", "tfhe", "ckks", "bgv", "bfv", "encrypted compute".
---

# Homomorphic Encryption Implementation

## Purpose
This skill guides cryptographic engineers and data scientists in designing systems that compute on encrypted data (Fully Homomorphic Encryption - FHE). It covers scheme selection, noise management, and performance optimization.

## When to Use
- Outsoring computation to untrusted cloud providers without exposing plaintext data.
- Privacy-preserving machine learning (PPML) inference.
- Secure multi-party computation involving encrypted sets.
- Implementing CKKS, BFV, BGV, or TFHE schemes.

## Key Information

### 1. FHE Schemes
*   **CKKS (Cheon-Kim-Kim-Song)**: Best for real-number arithmetic (floating point) and machine learning. Operations are approximate (contains rounding errors).
*   **BFV/BGV**: Best for exact integer arithmetic. Suitable for financial calculations or strict deterministic logic.
*   **TFHE (Fast Fully Homomorphic Encryption over the Torus)**: Best for boolean circuit evaluations (AND, OR, XOR). Fast bootstrapping makes it great for arbitrary logic and smart contracts (e.g., Zama's fhEVM).

### 2. The Noise Problem
Homomorphic encryption relies on Adding random "noise" to ciphertexts for security (Learning With Errors - LWE).
*   Every operation (especially multiplication) increases the noise in the ciphertext.
*   If noise exceeds the "noise budget", the data cannot be decrypted.
*   **Bootstrapping**: A computational trick to "refresh" a ciphertext, reducing its noise back to a nominal level, allowing for *infinite* computations (Fully Homomorphic). Bootstrapping is highly computationally expensive.

### 3. SIMD / Batching
To make FHE practical, operations must be vectorized.
*   **Polynomial Slots**: In schemes like CKKS/BFV, a single ciphertext is a polynomial that can hold an array of values (e.g., 4096 or 8192 slots).
*   A single FHE multiplication multiplies all 4096 slots simultaneously (SIMD).
*   Algorithms must be rewritten to leverage vector mathematics and rotations instead of standard arrays and indices.

### 4. Tooling & Compilers
*   **Microsoft SEAL**: Standard C++ library (with Python wrappers) for BFV, BGV, and CKKS.
*   **Zama Concrete**: Rust/Python framework making TFHE accessible, especially for compiling standard Python ML models into FHE circuits.
*   **OpenFHE**: Multi-scheme library supporting almost all post-quantum and FHE primitives.

## Best Practices
- ✅ **Minimize Multiplicative Depth**: Rewrite algorithms to use the shallowest possible tree of multiplications to delay or avoid bootstrapping.
- ✅ **Use Polynomial Batching**: Never encrypt a single integer in a ciphertext; always pack arrays of data into slots.
- ✅ **Approximation over Branching**: FHE cannot do standard `if/else` branching (since the server doesn't know the boolean result). Use polynomial approximations (like Taylor series) for activation functions like ReLU or Sigmoid.
- ❌ **Avoid Arbitrary Control Flow**: You cannot use data-dependent loops (e.g., `while(encrypted_x > 0)`). Loops must have a static, deterministic bound.
