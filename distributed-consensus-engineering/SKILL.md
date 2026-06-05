---
name: distributed-consensus-engineering
description: Deep dive into implementing and optimizing distributed consensus protocols like Paxos, Raft, and Byzantine Fault Tolerance (BFT). Triggers on "raft protocol", "paxos", "bft consensus", "hotstuff", "leader election", "split brain".
---

# Distributed Consensus Engineering

## Purpose
This skill provides advanced guidance for systems engineers building distributed databases, state machines, or blockchain networks that require strong consistency and fault tolerance via consensus protocols.

## When to Use
- Implementing leader election and log replication.
- Building distributed key-value stores (like etcd, ZooKeeper).
- Designing Byzantine Fault Tolerant (BFT) systems for trustless networks.
- Debugging split-brain scenarios or network partition recoveries.

## Key Information

### 1. Crash Fault Tolerance (CFT) vs. Byzantine Fault Tolerance (BFT)
*   **CFT (Paxos, Raft)**: Assumes nodes may crash or networks may drop packets, but nodes do not maliciously lie. Requires $2f + 1$ nodes to survive $f$ failures (e.g., 3 nodes to survive 1 crash).
*   **BFT (PBFT, HotStuff, Narwhal)**: Assumes nodes can be actively malicious or compromised. Requires $3f + 1$ nodes to survive $f$ malicious nodes (e.g., 4 nodes to survive 1 traitor).

### 2. Raft Protocol Mechanics
Raft decomposes consensus into three independent sub-problems:
*   **Leader Election**: Randomized timers prevent split votes. If a follower hears no heartbeat, it becomes a candidate.
*   **Log Replication**: The leader accepts client commands, appends to its log, and issues `AppendEntries` RPCs to followers. A log is committed once safely replicated to a majority.
*   **Safety**: If a leader crashes, the new leader is guaranteed to hold all committed entries.

### 3. Advanced BFT Protocols (HotStuff / DAGs)
*   **PBFT**: $O(N^2)$ message complexity. Doesn't scale well beyond ~100 nodes.
*   **HotStuff**: Used in modern blockchains (Aptos, Libra/Diem). Achieves linear $O(N)$ message complexity using Threshold Signatures and a rotating leader.
*   **DAG-based Consensus (Narwhal, Bullshark)**: Separates data dissemination (mempool) from metadata ordering (consensus), achieving massive throughput by eliminating the leader bottleneck for data transfer.

### 4. Edge Cases to Handle
*   **Split Brain**: When a network partitions, ensuring only one partition can achieve a quorum.
*   **Stale Leaders**: A partitioned leader might not realize it's deposed. It must be prevented from serving stale reads (e.g., by requiring a heartbeat quorum for reads, as in Raft).
*   **Log Compaction**: Logs grow infinitely. Implement snapshotting to discard historical logs safely.

## Best Practices
- ✅ **Use Existing Implementations**: Unless doing research or building a bespoke blockchain, use established libraries (e.g., Hashicorp's Raft, `etcd/raft`) rather than rolling your own consensus.
- ✅ **Test with Jepsen**: Validate your distributed system using Jepsen tests to inject network partitions, clock skews, and node crashes.
- ✅ **Idempotency**: Ensure that client commands applied to the state machine are idempotent, as network retries can cause duplicates.
- ❌ **Avoid Dual-Writes**: Never write to a database and then publish to a queue outside of the consensus log. Use the consensus log as the absolute source of truth (Event Sourcing).
