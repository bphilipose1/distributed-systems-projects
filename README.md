# Distributed Systems Projects

A collection of distributed systems implementations built in Python, covering core concepts from peer-to-peer networking, consensus algorithms, and real-time data processing. These projects were developed for CPSC 5520 (Distributed Systems) at Seattle University.

---

## Table of Contents

- [Overview](#overview)
- [Projects](#projects)
  - [Bitcoin P2P Node & Blockchain Integrity](#1-bitcoin-p2p-node--blockchain-integrity)
  - [Bully Algorithm Leader Election](#2-bully-algorithm-leader-election)
  - [Chord Distributed Hash Table](#3-chord-distributed-hash-table)
  - [Real-Time Forex Arbitrage Detector](#4-real-time-forex-arbitrage-detector)
- [Distributed Systems Concepts Demonstrated](#distributed-systems-concepts-demonstrated)
- [Learning Outcomes](#learning-outcomes)
- [References](#references)

---

## Overview

This repository explores the practical implementation of foundational distributed systems algorithms and protocols. Each project targets a distinct problem domain: blockchain integrity, fault-tolerant leader election, decentralized key-value storage, and real-time graph-based anomaly detection.

All projects use Python's standard library (no third-party frameworks) and communicate over raw TCP or UDP sockets, making the protocol mechanics explicit and observable.

---

## Projects

### 1. Bitcoin P2P Node & Blockchain Integrity

**Directory:** `block_chain/`

Connects to a live Bitcoin mainnet node, navigates the P2P protocol, retrieves a specific block, and simulates how the network would reject a tampered block.

#### What It Does

- Performs the Bitcoin version/verack handshake with a real peer node
- Sends `getblocks` messages to walk the chain inventory
- Requests a target block via `getdata` and parses its full structure (header, transactions, inputs, outputs)
- Modifies the first transaction (increments the first byte), recomputes the Merkle root, and demonstrates proof-of-work invalidation

#### Tech Stack

| Component | Detail |
|-----------|--------|
| Language | Python 3 |
| Protocol | Bitcoin P2P protocol (port 8333) |
| Transport | TCP (`socket`) |
| Hashing | SHA-256 double-hash (`hashlib`) |
| Encoding | Little-endian binary, CompactSize integers |

#### Key Concepts

- **Bitcoin wire protocol:** Magic bytes, 24-byte message headers, checksum verification
- **Merkle trees:** Pairwise SHA-256 hashing of transaction hashes to produce a block's Merkle root
- **Proof-of-work:** Block hash must be below a difficulty target; any transaction change invalidates both the Merkle root and the PoW
- **Endianness handling:** Bitcoin uses little-endian for most fields but big-endian for hashes displayed to users

#### How to Run

```bash
# Requires a reachable Bitcoin node (default: 5.14.1.135:8333)
cd block_chain
python3 lab5.py
```

The target block index is derived from `SU_ID % 10000`. Output includes parsed block headers, all transactions with inputs/outputs, the Merkle root recomputation, and the simulated rejection message.

#### Example Output (excerpt)

```
Connected to node 5.14.1.135:8333

SENDING MESSAGE
(85) f9beb4d9...
  HEADER
    magic: f9beb4d9
    command: version
    payload size: 61

Block Header:
Version: 70015
Previous Block Hash: 000000000000000000...
Merkle Root: a3f2...
Nonce: 3849201

Flipping First Transaction's First Bit
Old Transaction: 01000000...
Modified Transaction: 02000000...

New Merkle Root: b7c1...
Work Rejected: Invalid Merkle root, it should be matching the block header.
Work Rejected: Proof-of-work invalid. The block hash does not meet the difficulty target.
```

---

### 2. Bully Algorithm Leader Election

**Directory:** `bully_algorithm/`

A multi-node, multi-threaded implementation of the Bully Algorithm for leader election in a distributed system. Nodes register with a Group Coordination Daemon (GCD), elect a leader, and handle leader failure and recovery automatically.

#### What It Does

- Spawns multiple nodes, each with a unique identity `(days_to_moms_birthday, SU_ID)`
- Nodes register with a GCD to receive the member list
- Implements the full bully election cycle: `ELECTION` -> `OK` -> `COORD`
- Periodically probes the leader; triggers a new election if the leader stops responding
- Supports simulated leader failure (`play_dead`) and recovery: the failed node drops all requests, then re-registers and forces a new election when it comes back

#### Tech Stack

| Component | Detail |
|-----------|--------|
| Language | Python 3 |
| Transport | TCP (`socket`, `socketserver.ThreadingTCPServer`) |
| Serialization | `pickle` |
| Concurrency | `threading`, `queue.Queue`, `threading.Event`, `threading.Lock` |

#### Key Concepts

- **Bully Algorithm:** The node with the highest identity wins elections. Any node can call an election; the highest responder suppresses lower nodes and declares itself coordinator.
- **Fault tolerance:** Followers probe the leader at random intervals (0.5-3s). No OK within timeout means the leader is presumed dead.
- **Producer/consumer threading:** Listener threads push to `recv_queue`; a handler thread consumes it. Outbound messages go through `sender_queue` processed by a dedicated sender thread.
- **State machine:** Nodes transition through `STARTUP -> CANDIDATE -> TRASH_CANDIDATE -> FOLLOWER/LEADER`. `TRASH_CANDIDATE` decays back to `CANDIDATE` after 10s if no `COORD` is received.
- **Thread safety:** All shared state (`leader_ID`, `node_state`, `members`) protected with `threading.Lock` or `threading.Event`.

#### How to Run

```bash
# Start the GCD first (requires gcd2.py, not included here)
python3 gcd2.py 50040

# Then start the nodes
cd bully_algorithm
python3 bully_algo.py cs2.seattleu.edu 50040
```

`main()` automatically spawns three nodes and runs the failure simulation loop. Node states and leader views are printed with timestamps after each event.

#### Example Output (excerpt)

```
[2024-10-14 10:23:01] HELLO I AM (61, 4140754)
[2024-10-14 10:23:01] (61, 4140754): Starting Election...
[2024-10-14 10:23:01] (61, 4140754): I AM CANDIDATE
(295, 5029381): I view (295, 5029381) as LEADER!
[2024-10-14 10:23:07] (295, 5029381): I AM SUPREME LEADER

LEADER IS HIT BY A STORM AND OFFLINE
[2024-10-14 10:23:11] (154, 3201983): No OK response, leader assumed dead. Contacting GCD.
[2024-10-14 10:23:11] (154, 3201983): I AM SUPREME LEADER
```

---

### 3. Chord Distributed Hash Table

**Directory:** `chord_dht/`

A complete implementation of the Chord DHT protocol, including finger table initialization, key routing via `find_successor`, dynamic node joining, key transfer, and an NFL quarterback stats dataset stored across the ring.

#### What It Does

- Initializes a Chord ring with configurable `M` (ring size `2^M`)
- Nodes join by contacting a known peer, initializing their finger table, updating all affected nodes, and claiming their key range
- Keys are hashed with SHA-1 and routed to the responsible node via O(log N) hops
- `chord_populate.py` reads `Career_Stats_Passing.csv` and distributes rows as key-value pairs across the network
- `chord_query.py` retrieves a player's stats by `player_id + year` key from any node in the ring

#### Tech Stack

| Component | Detail |
|-----------|--------|
| Language | Python 3 |
| Transport | TCP (`socket`) |
| Serialization | `pickle` |
| Hashing | SHA-1 (`hashlib`) |
| Concurrency | `threading` |
| Data | NFL QB career passing stats (CSV) |

#### Key Concepts

- **Chord protocol (Stoica et al., 2001):** Consistent hashing maps both nodes and keys to a circular identifier space. Each node maintains an M-entry finger table for O(log N) lookup.
- **Finger tables:** Entry `i` of node `n` points to the first node at or after position `n + 2^(i-1)` on the ring.
- **Dynamic joining:** A new node finds its successor, initializes its finger table via RPCs, calls `update_others()` to fix all nodes that should now point to it, and pulls its key range from its successor.
- **RPC over TCP:** All inter-node calls (`find_successor`, `find_predecessor`, `update_finger_table`, `put_value`, `get_value`, `transfer_keys`) are implemented as serialized TCP messages.
- **ModRange:** A modular arithmetic range class handles wrap-around at the ring boundary correctly.

#### How to Run

```bash
cd chord_dht

# Start the first node (no existing network)
python3 chord_node.py 34000

# Join more nodes to the ring (provide an existing node's port)
python3 chord_node.py 34001 34000
python3 chord_node.py 34002 34000

# Wait for nodes to print heartbeat messages, then populate
python3 chord_populate.py 34000 Career_Stats_Passing.csv 100

# Query a player's stats (player_id + year as key)
python3 chord_query.py 34001 <player_id> <year>
```

> Note: Use ports from `TEST_PORTS` to avoid SHA-1 hash collisions in the node map at `M=7`.

#### Example Output (heartbeat)

```
Heartbeat <34: [67,89,34,45,67,89,34]34>
Storage Size: 12

Keys:
[3, 7, 14, 22, 31, 45, 67, 72, 89, 91, 103, 120]
```

---

### 4. Real-Time Forex Arbitrage Detector

**Directory:** `real_time_forex_arbitrage_detector/`

Subscribes to a live UDP forex quote stream, maintains a directed graph of exchange rates, and applies the Bellman-Ford algorithm to detect arbitrage cycles (negative-weight cycles in log-space) in real time.

#### What It Does

- Sends a UDP subscription message to a forex quote provider
- Listens for incoming quote packets on a dedicated UDP port, deserializing binary-encoded quote batches
- Updates a directed graph with each new quote, using `log10(rate)` as edge weights (so a profitable cycle becomes a negative-weight cycle)
- Removes stale quotes (older than 1.5s) from the graph to keep the market picture current
- Runs Bellman-Ford after every update and prints complete arbitrage paths with per-step exchange rates and profit when a cycle is found

#### Tech Stack

| Component | Detail |
|-----------|--------|
| Language | Python 3 |
| Transport | UDP (`socket`) |
| Concurrency | `threading`, `queue.Queue` |
| Algorithm | Bellman-Ford (negative cycle detection) |
| Math | `math.log10` for weight transformation |

#### Key Concepts

- **Arbitrage as negative cycles:** If `log(r_ab) + log(r_bc) + log(r_ca) < 0`, then `r_ab * r_bc * r_ca > 1`, meaning the round-trip yields profit. Bellman-Ford detects this directly.
- **Stale data management:** Quotes expire after 1.5s. Edges are removed from the graph when their timestamp exceeds the validity window.
- **Out-of-sequence handling:** Each quote's timestamp is compared to the last-seen timestamp for that currency pair; older quotes are silently ignored.
- **Producer/consumer pipeline:** A listener thread pushes deserialized quote batches to a `queue.Queue`; the main loop processes them sequentially and calls `check_arbitrage()` after each batch.
- **Bellman-Ford tolerance:** A small epsilon (`1e-12`) prevents floating-point rounding errors from producing false positive arbitrage signals.

#### How to Run

```bash
cd real_time_forex_arbitrage_detector

# Requires a running forex provider (e.g., provided by course infrastructure)
python3 lab3.py <provider_host> <provider_port>

# Example:
python3 lab3.py localhost 50040
```

#### Example Output

```
Subscribed to Forex Provider at localhost:50040
2024-10-26 22:38:30.081998 USD JPY 99.97254180908203
2024-10-26 22:38:30.081998 EUR USD 1.09953999519348
2024-10-26 22:38:32.084253 AUD CAD 0.30274885892868
Removing stale quote for EUR/JPY
ARBITRAGE:
    start with USD 100
    exchange USD for GBP at 0.7998848127301004 --> GBP 79.98848127301004
    exchange GBP for CAD at 0.8257669438777088 --> CAD 66.05184372623283
    exchange CAD for AUD at 3.303067775510835  --> AUD 218.1737165251972
    exchange AUD for USD at 0.7499600052833557 --> USD 163.62156159792625
```

---

## Distributed Systems Concepts Demonstrated

| Concept | Project(s) |
|---------|-----------|
| Peer-to-peer networking & wire protocols | Blockchain |
| Cryptographic integrity (Merkle trees, SHA-256) | Blockchain |
| Proof-of-work consensus | Blockchain |
| Leader election & fault tolerance | Bully Algorithm |
| Distributed state machine | Bully Algorithm |
| Thread-safe concurrent message passing | Bully Algorithm, Chord DHT, Forex |
| Consistent hashing | Chord DHT |
| Distributed key-value storage | Chord DHT |
| Remote procedure calls (RPC) over TCP | Chord DHT |
| O(log N) routing with finger tables | Chord DHT |
| Real-time graph construction from live data | Forex Arbitrage |
| Negative-cycle detection (Bellman-Ford) | Forex Arbitrage |
| Stale data expiry & out-of-sequence rejection | Forex Arbitrage |
| UDP pub/sub messaging | Forex Arbitrage |

---

## Learning Outcomes

1. **Protocol implementation from scratch:** All four projects implement networking protocols at the byte level, without protocol libraries, making framing, serialization, and error handling explicit.

2. **Concurrency design patterns:** The bully algorithm and forex detector both use producer/consumer queues with threading locks and events, demonstrating how to build correct concurrent systems in Python.

3. **Trade-offs in consistency vs. availability:** The Chord DHT shows how consistent hashing distributes load while the join/key-transfer protocol demonstrates the cost of maintaining consistency when the topology changes.

4. **Graph algorithms in a distributed context:** Using Bellman-Ford on a live, continuously-mutating graph illustrates how classical algorithms adapt to real-time, incomplete, and stale data.

5. **Failure detection and recovery:** The bully algorithm project shows how timeout-based failure detectors and re-election protocols restore system liveness after a node goes down, including the subtlety of preventing split-brain scenarios.

---

## References

- **Bully Algorithm:** Garcia-Molina, H. (1982). *Elections in a Distributed Computing System.* IEEE Transactions on Computers, C-31(1), 48-59.
- **Chord DHT:** Stoica, I., Morris, R., Karger, D., Kaashoek, M. F., & Balakrishnan, H. (2001). *Chord: A Scalable Peer-to-peer Lookup Service for Internet Applications.* ACM SIGCOMM.
  - [Paper PDF](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf)
- **Bellman-Ford Algorithm:** Bellman, R. (1958). *On a Routing Problem.* Quarterly of Applied Mathematics, 16(1), 87-90.
- **Forex Arbitrage via Negative Cycles:** Sedgewick, R. & Wayne, K. *Algorithms, 4th Edition.* Section 4.4 (Shortest Paths).
- **Bitcoin P2P Protocol:** Bitcoin Developer Documentation. [https://developer.bitcoin.org/reference/p2p_networking.html](https://developer.bitcoin.org/reference/p2p_networking.html)
- **Bitcoin Merkle Trees:** Nakamoto, S. (2008). *Bitcoin: A Peer-to-Peer Electronic Cash System.* [https://bitcoin.org/bitcoin.pdf](https://bitcoin.org/bitcoin.pdf)
