---
name: tee-enclave-development
description: Expert guidance for confidential computing and building Trusted Execution Environments (TEEs) using AWS Nitro Enclaves, Intel SGX, and AMD SEV. Triggers on "nitro enclaves", "intel sgx", "confidential computing", "tee", "secure enclave", "attestation", "vsock".
---

# TEE Enclave Development

## Purpose
This skill provides guidance for building and deploying applications within Trusted Execution Environments (TEEs). It covers memory isolation, cryptographic attestation, and secure communication patterns for confidential computing.

## When to Use
- Processing highly sensitive data (e.g., PII, financial records, ML models) in the cloud.
- Building Multi-Party Computation (MPC) nodes.
- Protecting cryptographic keys from root-level access on the host.
- Implementing "Zero-Trust" architectures at the hardware level.

## Key Information

### 1. TEE Technologies
*   **AWS Nitro Enclaves**: Isolated virtual machines with no persistent storage, no interactive access (SSH), and only `vsock` communication.
*   **Intel SGX (Software Guard Extensions)**: Hardware-isolated "enclaves" within a process's address space. Requires complex memory management (PRM).
*   **AMD SEV (Secure Encrypted Virtualization)**: Encrypts the entire memory of a Virtual Machine using a hardware-managed key.

### 2. Attestation
Attestation is the process of proving to a remote party that your code is running inside a genuine, unmodified TEE.
*   **Measurement (PCR/MRENCLAVE)**: A cryptographic hash of the enclave's initial state (code, data, config).
*   **Quote**: A signed report containing the measurement, issued by the hardware security module (e.g., Intel Quoting Enclave).
*   **Verification**: The client verifies the quote against the hardware vendor's root certificate to establish trust.

### 3. Secure Communication (Vsock)
Nitro Enclaves use `vsock` (AF_VSOCK) instead of standard TCP/IP.
*   **Host-to-Enclave**: The host acts as a proxy for external traffic.
*   **Proxy Patterns**: Common to use a small Python/C proxy on the host to forward HTTPS traffic to the enclave's vsock port.

### 4. Development Workflow
1.  **Containerize**: Package your app in a standard Docker container.
2.  **Convert**: Convert the Docker image to an enclave image (e.g., `.eif` for Nitro).
3.  **Deploy**: Launch the enclave on a supported instance type.
4.  **Attest**: Perform the attestation handshake before sending secrets to the enclave.

### 5. Essential Tools
*   **`nitro-cli`**: Build and manage Nitro Enclaves.
*   **`Gramine` / `Occlum`**: Library OS (LibOS) tools that allow running unmodified Linux binaries inside Intel SGX.
*   **`Confidential Containers (CoCo)`**: CNCF project for running standard Kubernetes pods in TEEs.

## Best Practices
- ✅ **Stateless Design**: TEEs usually lose all data on restart; always store persistent encrypted data on external storage (S3/EBS) with keys retrieved via attestation.
- ✅ **Minimal TCB**: Keep the Trusted Computing Base (the code inside the enclave) as small as possible to reduce the attack surface.
- ✅ **Audit vsock Logic**: Ensure the host proxy doesn't leak sensitive data or allow unauthorized commands into the enclave.
- ❌ **Avoid Interactive Debugging**: TEEs are designed to block debugging; use structured logging (sent over vsock) and local TEE simulators for development.
