---
name: tla-plus-formal-verification
description: Expert guidance for modeling distributed systems, verifying safety/liveness properties, and debugging race conditions using TLA+, PlusCal, and the TLC model checker. Triggers on "tla+", "pluscal", "formal verification", "model checking", "distributed system modeling", "invariant", "liveness".
---

# TLA+ Formal Verification

## Purpose
This skill provides guidance for software architects and distributed systems engineers using TLA+ (Temporal Logic of Actions) and PlusCal to formally specify and verify the correctness of complex systems before writing implementation code.

## When to Use
- Designing distributed algorithms (e.g., Consensus, Replication).
- Identifying subtle race conditions or deadlocks in concurrent systems.
- Verifying business-critical state machines.
- Proving safety (nothing bad happens) and liveness (something good eventually happens) properties.

## Key Information

### 1. The Modeling Languages
*   **PlusCal**: An algorithm language that looks like pseudo-code (C/Python-like) and is translated into TLA+ for verification. Best for most software engineers.
*   **TLA+**: A mathematical language based on set theory and temporal logic. Used for the final specification and deep mathematical reasoning.

### 2. Core Concepts
*   **Variables**: Define the state of the system.
*   **Initial State (`Init`)**: The starting condition of the variables.
*   **Next State Relation (`Next`)**: The set of all possible transitions (actions) that can occur.
*   **Invariants**: Conditions that must ALWAYS be true (e.g., "At most one node is leader").
*   **Temporal Properties**: Conditions about sequences of states (e.g., "Eventually, a client receives a response").

### 3. Model Checking with TLC
TLC is the engine that explores the entire state space of your model to find violations.
*   **Safety Errors**: TLC provides a "Counter-example" trace showing exactly how the invariant was broken.
*   **Deadlock Detection**: Automatically checked by TLC unless disabled.
*   **Liveness**: Verified using temporal logic properties (e.g., `<>P`).

### 4. Verification Workflow
1.  **Define Scope**: What is the most critical part of the algorithm?
2.  **Write PlusCal**: Describe the algorithm processes and variables.
3.  **Define Invariants**: Translate "correctness" into boolean expressions.
4.  **Configure Model**: Set constants (e.g., `NumNodes = 3`) and run TLC.
5.  **Refine**: Fix bugs identified by counter-examples and re-run.

### 5. Common Patterns
*   **Processes**: Use `process` in PlusCal to model concurrent nodes or threads.
*   **Nondeterminism**: Use `either...or` or `with` to model unpredictable events (network drops, hardware failures).
*   **Refinement**: Checking if a low-level implementation model satisfies a high-level abstract specification.

## Best Practices
- ✅ **Start Small**: Keep state spaces manageable (e.g., 3 nodes, 2 messages) to avoid state explosion.
- ✅ **Focus on Transitions**: TLA+ is about *actions*. Model how the state changes, not just the state itself.
- ✅ **Iterative Modeling**: Model the "Happy Path" first, then add error states (packet loss, crashes).
- ❌ **Avoid Modeling Implementation Details**: Do not model low-level data structures like hash maps or specific library calls unless they are critical to the algorithm's logic.
