🛡️ VAULT
Fault-Tolerant Distributed Object Storage, Built to Survive Failure.

Store. Replicate. Detect. Repair. Recover.

A distributed object storage engine designed for unreliable nodes, network partitions, data corruption, and replica inconsistency.

🚀 What is Vault?

Vault is a fault-tolerant distributed object storage system that assumes hardware and networks will fail — and is designed to keep serving data anyway.

Instead of depending on one machine or one copy of a file, Vault distributes objects across multiple storage nodes and availability zones.

When something goes wrong — a node crashes, an availability zone becomes unreachable, a replica silently corrupts data, or the cluster experiences a network partition — Vault can:

🔀 Route around failed nodes
🧩 Split large objects into manageable chunks
📦 Replicate chunks across independent nodes
🧠 Maintain metadata consistency through Raft consensus
🔐 Verify data integrity using SHA-256
🌳 Detect replica divergence using Merkle trees
🩹 Automatically repair damaged or missing replicas
⚖️ Rebalance distributed storage
📊 Continuously expose cluster health and performance
The core idea

Failure is not an exception in Vault. Failure is a scenario the system is built to handle.

🎯 The Problem

Traditional file storage becomes difficult to reason about when storage is distributed.

A production-grade distributed storage system must answer questions like:

Failure / Challenge	Vault's Approach
💥 Storage node crashes	Replica-based failover
🌐 Network partition	Quorum + consensus
⚡ Concurrent operations	Distributed coordination
🧩 Large objects	64 MB chunking
🗄️ Uneven storage distribution	Consistent hashing
🦠 Silent data corruption	SHA-256 integrity verification
🔄 Replica divergence	Merkle-tree anti-entropy
📝 Metadata inconsistency	Raft replicated state machine
🩹 Missing replicas	Automatic background repair
🔥 Availability-zone failure	Cross-AZ replication
📈 Cluster growth	Rebalancing
🧪 Failure validation	Built-in chaos engineering

Vault brings these mechanisms together into one observable distributed-storage simulation.

🧠 Architecture at a Glance
                    ┌─────────────────────┐
                    │       CLIENT        │
                    │  Upload / Retrieve  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    OBJECT ROUTER    │
                    │  Consistent Hashing │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   OBJECT SHARDING   │
                    │     64 MB chunks    │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
       │    AZ-1     │  │    AZ-2     │  │    AZ-3     │
       │ Node-01     │  │ Node-03     │  │ Node-05     │
       │ Node-02     │  │ Node-04     │  │ Node-06     │
       └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                       N = 3 Replication
                               │
                               ▼
              ┌─────────────────────────────────┐
              │       INTEGRITY + CONSISTENCY   │
              │                                 │
              │  SHA-256 → Merkle Trees → Raft │
              └────────────────┬────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ SELF-HEALING ENGINE │
                    │  Detect → Recover   │
                    │  Repair → Rebalance │
                    └─────────────────────┘
🏗️ Cluster Topology

Vault currently visualizes a 6-node storage cluster distributed across 3 availability zones:

                    VAULT CLUSTER

       ┌──────────── AZ-1 ────────────┐
       │                              │
       │  Node-01      Node-02        │
       │                              │
       └──────────────────────────────┘

       ┌──────────── AZ-2 ────────────┐
       │                              │
       │  Node-03      Node-04        │
       │                              │
       └──────────────────────────────┘

       ┌──────────── AZ-3 ────────────┐
       │                              │
       │  Node-05      Node-06        │
       │                              │
       └──────────────────────────────┘

                  Replication
                      N = 3

This topology makes failure testing meaningful: replicas are not conceptually concentrated on a single machine or zone.

🔀 Consistent Hash Ring

Vault uses a consistent hashing ring to distribute the key space across storage nodes.

Instead of assigning objects using a simple sequential mapping:

Object A → Node 1
Object B → Node 2
Object C → Node 3

Vault maps object fingerprints onto a logical 360° ring:

                         Node-01
                           ●
                     ╱           ╲
                  ●                 ●
              Node-06             Node-02
                │                   │
                │      KEY SPACE    │
                │                   │
              Node-05             Node-03
                  ●                 ●
                     ╲           ╱
                           ●
                        Node-04
Why this matters

When nodes join or leave the cluster, consistent hashing minimizes the amount of data that needs to move.

That means:

Less unnecessary data movement
Faster rebalancing
Better scalability
More predictable storage distribution

Vault also uses virtual nodes to improve distribution across the key space.

🧩 Object Sharding

Large objects are not treated as one giant indivisible blob.

Vault splits objects into 64 MB binary chunks.

                    LARGE OBJECT
                         │
                         ▼
        ┌────────┬────────┬────────┬────────┐
        │ Chunk  │ Chunk  │ Chunk  │ Chunk  │
        │  01    │  02    │  03    │  04    │
        └────────┴────────┴────────┴────────┘
             │        │        │        │
             ▼        ▼        ▼        ▼
          Replicate across N = 3
             │        │        │
             ▼        ▼        ▼
          Node/AZ  Node/AZ  Node/AZ
Benefits
Parallel reads and writes
Smaller recovery units
Reduced repair bandwidth
Easier corruption detection
Better distribution across nodes
Large-object handling without treating the object as one storage unit
📦 Replication & Durability

Vault uses N=3 replication in the demonstrated cluster.

Each chunk can have three independent replicas:

                 CHUNK #42
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Replica A  Replica B  Replica C
        AZ-1        AZ-2        AZ-3

If one replica becomes unavailable, the remaining healthy replicas can continue participating in recovery.

The important design principle is:

Durability comes from independent copies, while availability comes from being able to route around failed copies.

🧠 Raft Consensus

Distributed storage is not only about storing bytes.

The system also needs a consistent view of cluster metadata and state.

Vault demonstrates a Raft-based consensus layer with:

👑 Current leader
🔢 Current term
🧾 Commit index
🗳️ Quorum voting
📜 Replicated log entries
🔄 Leader re-election after failure

Example cluster state exposed by the system:

RAFT CONSENSUS

Leader       : Node-01
Current Term : 14
Commit Index : #1048
Quorum       : Active
Why Raft?

If a leader suddenly disappears, the cluster should not simply continue making conflicting metadata decisions.

Vault can deliberately terminate the current Raft leader and demonstrate the resulting quorum-election flow.

🌳 Merkle Tree Anti-Entropy

One of the most important distributed-storage problems is silent corruption.

A disk can return data that is technically readable but no longer identical to the original data.

Vault addresses this through:

SHA-256 + Merkle Trees
                 Merkle Root
                     │
             ┌───────┴───────┐
             ▼               ▼
           Hash A           Hash B
          /     \           /     \
         ▼       ▼         ▼       ▼
      Chunk 1 Chunk 2   Chunk 3 Chunk 4

Instead of downloading every complete object from every replica, nodes can compare hierarchical hashes to locate divergence.

Vault's implementation uses Merkle comparison to identify inconsistent data and initiate repair.

🩹 Automatic Self-Healing

Detection is only half the problem.

Vault also demonstrates automatic recovery and replica repair.

        CORRUPTION / NODE FAILURE
                  │
                  ▼
          Integrity Verification
                  │
                  ▼
           Detect mismatch
                  │
                  ▼
            Find healthy
              replica
                  │
                  ▼
             Copy chunk
                  │
                  ▼
          Verify checksum
                  │
                  ▼
           Replica restored

When checksum discrepancies are detected, Vault can pull healthy chunks from the remaining replicas and restore the damaged data in the background.

The recovery philosophy

Detect → Isolate → Recover → Verify → Restore

This avoids treating recovery as a manual administrator task.

💥 Built-In Chaos Engineering

A major part of Vault is that failure scenarios are not theoretical.

The dashboard includes direct fault-injection controls.

1. Kill Raft Leader

Terminates the active Raft leader to trigger a quorum election.

Leader
  │
  ✕ CRASH
  │
  ▼
Quorum Election
  │
  ▼
New Leader
2. Partition an Availability Zone

Vault can isolate AZ-2, affecting Node-03 and Node-04 simultaneously.

       AZ-1              AZ-2              AZ-3
     ┌───────┐          ┌───────┐          ┌───────┐
     │ N1 N2 │          │ N3 N4 │          │ N5 N6 │
     └───────┘          └───╳───┘          └───────┘
        ✓                  PARTITION            ✓

This allows the system's behavior under partial network isolation to be observed.

3. Inject Silent Bit-Rot

Vault can deliberately corrupt random binary chunks without updating the corresponding state.

This creates a realistic integrity failure:

Healthy Replica
      │
      ├── SHA-256 ✓
      │
Corrupted Replica
      │
      └── SHA-256 ✗
               │
               ▼
         Merkle mismatch
               │
               ▼
          Auto-repair
4. Automatic Recovery

After introducing failures, the dashboard provides:

Run Automatic Recovery & Self-Healing

This allows the complete cluster recovery workflow to be demonstrated rather than merely described.

🔍 Object Explorer & Replica Mapper

Vault doesn't hide distributed storage behind a generic upload button.

The interface exposes the relationship between:

Object
  ↓
Chunks
  ↓
Checksums
  ↓
Replicas
  ↓
Storage Nodes
  ↓
Availability Zones

Objects are represented as chunks, checksummed with SHA-256, and replicated using the N=3 quorum model.

This makes the distributed-storage behavior observable instead of abstract.

📤 Upload Pipeline

The object upload flow is designed around the distributed architecture:

        SELECT OBJECT
              │
              ▼
        Determine size
              │
              ▼
       Split into 64 MB
           chunks
              │
              ▼
         SHA-256 hash
              │
              ▼
      Consistent hash
          placement
              │
              ▼
         N = 3 replicas
              │
              ▼
       Commit to cluster

The interface exposes filename, simulated file size, automatic quorum sharding, and the option to commit the object to the Vault cluster.

📊 Real-Time Cluster Telemetry

Vault provides a live operational view instead of leaving performance characteristics invisible.

The dashboard exposes metrics such as:

Metric	Demonstrated Value
⚡ Operations	14,250 ops/s
📖 Read latency P99	1.42 ms
✍️ Write latency P99	3.88 ms
🗄️ Storage overhead	3.0×
🟢 Availability SLA	99.999%

Note: These are the application's demonstrated/simulated dashboard metrics and should not be interpreted as an independent production benchmark.

📡 Event Log & Operational Visibility

Distributed systems are difficult to debug when internal state is invisible.

Vault therefore exposes a live operational event stream so that cluster activity can be observed while operations and failure scenarios occur.

The dashboard also provides controls for clearing the event log and observing system activity in real time.

🎓 Vault Explained — Without the Distributed-Systems Jargon

Vault intentionally includes a simplified architecture guide.

📖 1. Object Sharding

Think of a huge book.

Instead of putting the entire book on one machine, split it into chapters and distribute those chapters.

Vault uses 64 MB chunks for the same reason.

📦 2. Quorum Replication

Imagine keeping three independent copies of every important chapter.

If one machine disappears, the other copies can still provide the data.

Vault demonstrates this using N=3 replication.

🕐 3. Consistent Hashing

Imagine a 360° clock.

Every storage node owns a position on the clock.

An object's fingerprint determines where it belongs.

Adding or removing a node therefore doesn't require reshuffling the entire storage cluster.

🧬 4. Merkle Trees

Imagine every chapter having a fingerprint, and groups of fingerprints being combined into larger fingerprints.

If two replicas have different top-level fingerprints, the tree can be walked downward until the inconsistent chunk is found.

Vault then repairs that chunk from a healthy replica.

🔄 End-to-End Recovery Flow

The complete Vault philosophy can be summarized as:

             ┌──────────────┐
             │    CLIENT    │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │    SHARD     │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │   REPLICATE  │
             │     N = 3    │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │    VERIFY    │
             │   SHA-256    │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │   MONITOR    │
             └──────┬───────┘
                    │
          ┌─────────┴─────────┐
          │                   │
       HEALTHY             FAILURE
          │                   │
          ▼                   ▼
       Continue        Detect divergence
                              │
                              ▼
                       Merkle comparison
                              │
                              ▼
                         Find replica
                              │
                              ▼
                          Repair chunk
                              │
                              ▼
                       Verify checksum
                              │
                              ▼
                         Back to healthy
🧪 Failure Scenarios Demonstrated

Vault is designed around explicit failure scenarios rather than assuming a perfect infrastructure.

Scenario	What Vault Demonstrates
Node crash	Replica-based continuity and recovery
Raft leader crash	Leader re-election
Availability-zone isolation	Partial network failure handling
Silent bit-rot	Integrity verification
Replica inconsistency	Merkle-based divergence detection
Missing/corrupted chunks	Background replica repair
Node topology changes	Consistent-hash-based distribution
Large object upload	Chunking + distributed replication
Metadata changes	Raft-backed state consistency
💡 Why This Architecture?

Vault separates the distributed-storage problem into independent layers:

┌───────────────────────────────────────────────┐
│                OBJECT LAYER                   │
│ Upload • Retrieve • Shard • Map               │
├───────────────────────────────────────────────┤
│               STORAGE LAYER                   │
│ Nodes • Replicas • Availability Zones         │
├───────────────────────────────────────────────┤
│              CONSISTENCY LAYER                │
│ Raft • Quorum • Metadata                      │
├───────────────────────────────────────────────┤
│               INTEGRITY LAYER                 │
│ SHA-256 • Merkle Trees • Anti-Entropy         │
├───────────────────────────────────────────────┤
│                RECOVERY LAYER                 │
│ Repair • Rebalance • Self-Healing             │
├───────────────────────────────────────────────┤
│             OBSERVABILITY LAYER               │
│ Metrics • Events • Fault Injection            │
└───────────────────────────────────────────────┘

This makes it easier to reason about correctness, failure recovery, and operational behavior independently.

🏆 What Makes Vault Hackathon-Ready?
01 — Failure is Demonstrable

Instead of claiming that the system is fault tolerant, the dashboard provides controls to actively inject failures.

02 — Distributed Concepts Are Visual

Consistent hashing, topology, replicas, Raft state, Merkle trees, and telemetry are exposed through the interface.

03 — Recovery Is Part of the Product

The system doesn't stop at detecting a failure.

It demonstrates:

Detection → Recovery → Verification → Repair

04 — The Architecture Maps Directly to the Problem Statement
Requirement	Vault Component
Distributed storage	6-node cluster
Replication	N=3
Large volumes	64 MB chunking
Concurrent distributed operations	Quorum-oriented architecture
Node failures	Fault injection
Network partitions	AZ isolation
Data corruption	Bit-rot injection
Integrity verification	SHA-256
Replica inconsistency	Merkle anti-entropy
Metadata consistency	Raft
Automatic repair	Self-healing protocol
Rebalancing	Consistent hashing
Observability	Telemetry + event stream
🚀 Quick Demo — What Judges Should Try

If you're evaluating Vault, the fastest way to understand the project is:

Step 1 — Explore the cluster

Open the Cluster Topology and inspect the 6-node / 3-AZ architecture.

Step 2 — Upload an object

Go to Objects & Shards → upload an object → observe how the object is represented as chunks and replicated.

Step 3 — Kill the Raft leader

Open Chaos Engineering → trigger Leader Crash.

Watch the consensus state transition.

Step 4 — Partition AZ-2

Isolate the second availability zone and observe how the cluster reacts to partial failure.

Step 5 — Inject corruption

Trigger Silent Bit-Rot.

This intentionally creates inconsistent replica data.

Step 6 — Run recovery

Execute:

Automatic Recovery & Self-Healing

Observe the repair process.

Step 7 — Inspect Merkle synchronization

Open Anti-Entropy and trigger Merkle synchronization to understand how divergent replicas are identified.

🛠️ Core Technologies & Concepts

Vault is centered around distributed-systems primitives rather than a single framework.

Distributed Systems
Consistent Hashing
Virtual Nodes
Quorum Replication
Raft Consensus
Availability Zones
Failure Detection
Background Repair
Rebalancing
Data Integrity
SHA-256 Checksums
Merkle Trees
Anti-Entropy Synchronization
Replica Verification
Storage
Object Storage
64 MB Chunking
Replica Mapping
Distributed Placement
Reliability
Node Failure Injection
Leader Failure
Network Partition Simulation
Bit-Rot Simulation
Automatic Self-Healing
Observability
Cluster Topology
Consensus State
Event Logs
Throughput
P99 Latency
Storage Overhead
📈 Design Goals

Vault is designed around four primary goals:

             ┌──────────────────┐
             │    AVAILABILITY  │
             │  Keep serving    │
             │  through faults  │
             └────────┬─────────┘
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ DURABILITY │ │ CONSISTENCY│ │ EFFICIENCY │
│  Replicate │ │    Raft    │ │  Hashing   │
│  & Repair  │ │   + Quorum │ │ + Chunks   │
└────────────┘ └────────────┘ └────────────┘
Availability

Continue operating despite independently failing nodes and partial network failures.

Durability

Maintain multiple independent copies and repair damaged replicas.

Consistency

Coordinate metadata and cluster state through quorum-based consensus.

Efficiency

Reduce unnecessary data movement and recovery cost using chunking, consistent hashing, and Merkle-based comparison.

🔮 Future Improvements

Vault's architecture can be extended toward a production-grade storage platform with:

 Configurable replication factors
 Configurable quorum policies
 Erasure coding for lower storage overhead
 Persistent metadata storage
 Dynamic node joining/leaving
 Automatic load-aware rebalancing
 Read repair
 Write-ahead logging
 Snapshotting and Raft log compaction
 More granular failure domains
 Encryption at rest
 Authentication & authorization
 Prometheus/Grafana-style observability
 Persistent object versioning
 Multi-region replication
 Real-world benchmark suite
🌐 Live Demo
🚀 Try Vault

https://promptathon-eight.vercel.app/

💡 Tip for judges: Don't just browse the dashboard — use the Chaos Engineering controls. Vault is designed to be understood by breaking it.

🧭 Project Philosophy
"A distributed storage system should not merely store data. It should know when that data is no longer safe — and know how to make it safe again."

Vault combines distribution, replication, consensus, integrity verification, anti-entropy, and automated recovery into one interactive system.

The result is a storage architecture where failure becomes something that can be observed, reasoned about, injected, detected, and recovered from.

⭐ Vault in One Sentence

Vault is a distributed object storage engine that deliberately expects nodes, networks, and data to fail — then uses replication, Raft, consistent hashing, SHA-256, Merkle trees, and automated self-healing to keep the cluster operational.

🚀 Launch Live Demo
