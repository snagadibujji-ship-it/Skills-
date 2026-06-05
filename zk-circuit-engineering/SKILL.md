---
name: zk-circuit-engineering
description: Expert guidance for Zero-Knowledge Proofs (ZKPs), cryptographic circuit design using Circom or Halo2, and proving systems (SNARKs, STARKs). Triggers on "zkp", "zero-knowledge", "snark", "stark", "circom", "halo2", "arithmetization".
---

# Zero-Knowledge Circuit Engineering

## Purpose
This skill provides advanced guidance for cryptographic engineers building Zero-Knowledge Proof (ZKP) systems. It covers the shift from traditional programming to arithmetic circuit design, optimization of constraints, and the nuances of different proving backends.

## When to Use
- Designing circuits for privacy-preserving applications (e.g., anonymous voting, private transactions).
- Scaling blockchain throughput via ZK-Rollups.
- Auditing Circom or Halo2 code for under-constrained variables.
- Optimizing polynomial commitment schemes (PCS) and arithmetization.

## Key Information

### 1. The Paradigm Shift: Arithmetization
In ZK, you don't write imperative code; you define a mathematical system of constraints (equations) that a valid computation must satisfy.
*   **R1CS (Rank-1 Constraint System)**: Used by Circom. Constraints take the form `A * B = C`.
*   **PLONKish / Halo2**: Uses custom gates and lookup tables, allowing for more flexible and efficient representations of complex operations (like hash functions).
*   **AIR (Algebraic Intermediate Representation)**: Used heavily in STARKs, focusing on execution trace tables and transition constraints.

### 2. Proving Systems
*   **SNARKs (e.g., Groth16, PLONK)**: Small proof sizes, fast verification. Often require a Trusted Setup (Groth16) or Universal Setup (PLONK).
*   **STARKs**: Larger proofs, but no trusted setup, quantum-resistant (rely on hash functions), and highly scalable for massive computations.

### 3. Circuit Design with Circom (R1CS)
*   **Signals**: `in`, `out`, and internal signals.
*   **Constraints (`<==`, `===`)**: `a <== b * c` both assigns the value of `b * c` to `a` AND generates an R1CS constraint `a = b * c`.
*   **Non-deterministic Computation**: ZK circuits can use "hints" (unconstrained computation) to find a solution (e.g., division is just multiplication in reverse), but the result MUST be constrained (`a * b === 1` to prove `b` is the inverse of `a`).

### 4. Circuit Design with Halo2 (PLONKish)
*   **Columns & Rows**: Data is arranged in a matrix.
*   **Custom Gates**: Define specific polynomial relationships between cells (e.g., `s_mul * (a * b - c) = 0`).
*   **Lookup Tables**: Precompute expensive operations (like bitwise XOR or range checks) and "look up" the result in the circuit, saving constraints.

### 5. Common Vulnerabilities
*   **Under-constrained Circuits**: A signal is assigned a value but not constrained. A malicious prover can forge a proof with a fake value.
*   **Missing Range Checks**: If you don't constrain a variable to be within a certain bit size (e.g., 64-bit), it can wrap around the finite field prime ($p$), leading to exploits.

## Best Practices
- ✅ **Test the "Bad" Path**: Write tests that intentionally provide invalid witness data and assert that the prover fails.
- ✅ **Use Lookup Tables (Halo2)**: For non-native arithmetic (like SHA256 or Keccak), use PLONKish lookup tables rather than raw gates.
- ✅ **Audit External Templates**: Treat standard libraries (like `circomlib`) as highly trusted, but audit any community-provided templates for missing constraints.
- ❌ **Avoid Conditionals in Circuits**: `if/else` statements do not exist in math. You must compute BOTH branches and use a multiplexer (`out = condition * true_branch + (1 - condition) * false_branch`).
