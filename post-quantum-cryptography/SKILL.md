---
name: post-quantum-cryptography
description: Expert guidance for implementing and migrating to Post-Quantum Cryptography (PQC) algorithms. Triggers on "post-quantum", "pqc", "crystals-kyber", "ml-kem", "dilithium", "quantum-safe", "sphincs+".
---

# Post-Quantum Cryptography (PQC)

## Purpose
This skill provides guidance for cryptographic engineers and security architects tasked with securing systems against future quantum computer attacks. It covers NIST-standardized algorithms, implementation pitfalls, and migration strategies.

## When to Use
- Preparing long-lived data for "Harvest Now, Decrypt Later" threats.
- Migrating TLS, SSH, or code-signing infrastructure to quantum-safe standards.
- Implementing the NIST PQC standards (FIPS 203, 204, 205).
- Designing hybrid cryptographic schemes (Classical + Quantum-Safe).

## Key Information

### 1. The NIST PQC Standards (2024 Finalized)
*   **ML-KEM (formerly CRYSTALS-Kyber)**: Module-Lattice-based Key-Encapsulation Mechanism. Preferred for key exchange and encryption. (FIPS 203)
*   **ML-DSA (formerly CRYSTALS-Dilithium)**: Module-Lattice-based Digital Signature Algorithm. General-purpose digital signatures. (FIPS 204)
*   **SLH-DSA (formerly SPHINCS+)**: Stateless Hash-based Digital Signature Algorithm. Extremely robust but larger signatures and slower. (FIPS 205)

### 2. Hybrid Cryptography
To maintain security against today's threats while preparing for tomorrow's, use hybrid schemes.
*   **X25519 + ML-KEM**: Combine Elliptic Curve Diffie-Hellman with Kyber.
*   **RSA/ECDSA + ML-DSA**: Dual-sign certificates to ensure compatibility with legacy systems while providing quantum resistance.

### 3. Implementation Pitfalls
*   **Side-Channel Attacks**: Lattice-based algorithms are highly susceptible to timing and power analysis. Use implementations with constant-time modular arithmetic.
*   **Key/Signature Size**: PQC keys and signatures are significantly larger than classical counterparts (e.g., Kyber-768 public key is ~1KB vs X25519's 32B). Ensure buffers and protocols (like MTU limits) can handle the increase.
*   **Parameter Selection**: Always use NIST-recommended levels (e.g., Level 3 for AES-192 equivalent).

### 4. Migration Strategy (PQ-Readiness)
1.  **Inventory**: Identify all classical asymmetric cryptography (RSA, ECC) in your infrastructure.
2.  **PQ-Agility**: Update protocols to support negotiation of multiple algorithm types.
3.  **Hybrid Deployment**: Phase in hybrid key exchange first for network traffic.
4.  **Full Migration**: Replace classical signatures for cold storage and code signing.

### 5. Essential Libraries
*   **`liboqs` (Open Quantum Safe)**: The industry-standard C library for PQC algorithms.
*   **`OQS-OpenSSL`**: Integration of liboqs into OpenSSL.
*   **`Go crypto/tls`**: Recent versions (1.23+) include experimental support for X25519MLKEM768.

## Best Practices
- ✅ **Use Hybrid by Default**: Never rely solely on PQC algorithms during the early transition phase; always combine them with a trusted classical algorithm.
- ✅ **Verify Constant-Time**: Audit low-level math for variable-time branches or memory lookups.
- ✅ **Check Buffer Limits**: Ensure your TLS stack or database schema can accommodate 1KB-3KB keys and signatures.
- ❌ **Avoid "Rolling Your Own"**: PQC math (polynomial multiplication over rings) is complex and easy to get wrong; use verified libraries from the Open Quantum Safe project.
