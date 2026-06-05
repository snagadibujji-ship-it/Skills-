---
name: ebpf-observability-engineering
description: Expert guidance for writing, debugging, and deploying eBPF programs for Linux kernel observability, networking (XDP), and security tracing. Triggers on "ebpf", "xdp", "libbpf", "kernel tracing", "bpf maps", "kprobe", "uprobe".
---

# eBPF Observability Engineering

## Purpose
This skill provides advanced guidance for systems engineers working with eBPF (extended Berkeley Packet Filter). It covers the full lifecycle from writing C/Rust BPF programs to loading them via `libbpf` or `bcc` and visualizing kernel-level telemetry.

## When to Use
- Implementing kernel-level tracing or profiling.
- Building high-performance networking hooks via XDP.
- Monitoring system calls or network packets with minimal overhead.
- Debugging performance bottlenecks in the Linux kernel or userspace.

## Key Information

### 1. BPF Program Types
*   **`BPF_PROG_TYPE_KPROBE`**: Trace kernel function entry/exit.
*   **`BPF_PROG_TYPE_TRACEPOINT`**: Stable kernel hooks for tracing.
*   **`BPF_PROG_TYPE_XDP`**: Express Data Path for fast packet processing at the driver level.
*   **`BPF_PROG_TYPE_SOCKET_FILTER`**: Monitor and filter network traffic.

### 2. Development Workflow (libbpf-first)
Prefer `libbpf` over `bcc` for production due to BPF CO-RE (Compile Once – Run Everywhere).
1.  **C Code**: Write the BPF program (usually `*.bpf.c`).
2.  **Clang/LLVM**: Compile to BPF bytecode: `clang -target bpf -g -O2 -c myprog.bpf.c -o myprog.bpf.o`.
3.  **Skeleton**: Generate the BPF skeleton for your userspace loader: `bpftool gen skeleton myprog.bpf.o > myprog.skel.h`.
4.  **Userspace**: Write the C/C++/Rust loader that attaches the program to the target hook.

### 3. Safety & Verification
BPF programs are verified by the kernel before execution.
*   **Bounded Loops**: All loops must be provably finite.
*   **Memory Access**: Always use `bpf_probe_read_kernel()` for safe memory access.
*   **Stack Limit**: The stack is limited to 512 bytes. Use BPF maps for larger data.

### 4. BPF Maps
Maps are the primary way to share data between the kernel and userspace.
*   **`BPF_MAP_TYPE_HASH`**: Key-value pairs.
*   **`BPF_MAP_TYPE_ARRAY`**: Index-based storage.
*   **`BPF_MAP_TYPE_RINGBUF`**: High-performance buffer for event streaming to userspace (preferred over PERF_EVENT_ARRAY).

### 5. Essential Tools
*   **`bpftool`**: The Swiss Army knife for BPF (inspect programs, maps, and attachments).
*   **`veristat`**: Test the BPF verifier against your program.
*   **`btfhub`**: For BTF (BPF Type Format) support on older kernels.

## Best Practices
- ✅ **Use CO-RE**: Always leverage `vmlinux.h` and BTF-based relocations.
- ✅ **Minimize Overhead**: Avoid heavy logic in high-frequency hooks (like packet-level XDP).
- ✅ **Ring Buffer**: Use `BPF_MAP_TYPE_RINGBUF` for event delivery to avoid dropped packets and multi-CPU contention.
- ❌ **Avoid Global Variables**: Use maps or specialized BPF global data sections (`.rodata`, `.data`).
