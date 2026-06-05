---
name: compiler-backend-optimization
description: Advanced guidance for LLVM infrastructure, writing optimization passes, Just-In-Time (JIT) compilation, and intermediate representations. Triggers on "llvm pass", "compiler optimization", "jit compilation", "intermediate representation", "ir builder", "ssa form".
---

# Compiler Backend & LLVM Optimization

## Purpose
This skill is designed for language creators and performance engineers working on compiler backends. It provides expert guidance on manipulating Intermediate Representation (IR), writing custom LLVM optimization passes, and generating optimal machine code.

## When to Use
- Writing a custom compiler frontend that targets LLVM IR.
- Implementing custom peephole optimizations or Loop invariant code motion (LICM).
- Building Just-In-Time (JIT) compilers for dynamic languages or DSLs (e.g., using ORC JIT).
- Debugging code generation or register allocation issues.

## Key Information

### 1. LLVM IR and SSA Form
LLVM uses a strictly typed, RISC-like Intermediate Representation.
*   **Static Single Assignment (SSA)**: Every variable (register) is assigned exactly once. This makes data flow analysis trivial.
*   **Phi Nodes (`phi`)**: Because variables can only be assigned once, control flow joins (like the end of an `if/else`) require Phi nodes to determine which value to use based on the basic block that preceded it.
*   **Basic Blocks**: A sequence of instructions with a single entry point and a single exit point (terminator instruction like `ret` or `br`).

### 2. Writing LLVM Passes
Optimization passes transform the IR to make it faster or smaller.
*   **New Pass Manager**: Always use the new Pass Manager framework (`llvm::PassInfoMixin`).
*   **Analysis vs. Transformation**: Analysis passes gather data (e.g., Dominator Tree, Target Library Info). Transformation passes mutate the IR.
*   Preserve analyses! If your pass mutates the IR, it must explicitly state which analyses it preserves, or LLVM will waste time recomputing them.

### 3. Instruction Selection (DAG)
The process of converting LLVM IR into target-specific machine instructions.
*   **SelectionDAG**: IR is converted into a Directed Acyclic Graph. Nodes are matched against target instructions using TableGen (`.td` files).
*   **GlobalISel**: The newer instruction selection framework designed to be faster and easier to test than SelectionDAG.

### 4. ORC JIT
The On-Request Compilation JIT API in LLVM.
*   Allows lazy compilation (compile only when a function is first called).
*   Manages memory allocation for generated machine code and links symbols dynamically.

## Best Practices
- ✅ **Use `IRBuilder`**: Always use `llvm::IRBuilder` to construct IR instructions, as it automatically performs constant folding (e.g., converting `add i32 2, 2` directly to `4`).
- ✅ **Target Data Layout**: Ensure your Module is configured with the correct `DataLayout` for your target architecture (endianness, pointer sizes), or optimizations will fail or miscompile.
- ✅ **Leverage LLVM `opt`**: Use the standalone `opt` tool to test your passes in isolation using `.ll` text files before integrating them into a full compiler pipeline.
- ❌ **Avoid Memory for Locals**: Do not allocate variables using `alloca` for simple locals if you can avoid it. Use SSA registers. (If you must use `alloca`, use the `mem2reg` pass to promote them to registers).
