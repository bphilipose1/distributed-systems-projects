# Distributed Systems Projects

A collection of distributed systems implementations from CPSC 5520 at Seattle University. Each project demonstrates a core concept in distributed computing.

## Projects

| Project | Concept | Language |
|---------|---------|----------|
| [Blockchain](#blockchain) | Proof-of-work, Bitcoin protocol | Python |
| [Bully Algorithm](#bully-algorithm) | Leader election | Python |
| [Chord DHT](#chord-dht) | Distributed hash table (P2P lookup) | Python |
| [Forex Arbitrage Detector](#real-time-forex-arbitrage-detector) | Graph algorithms, UDP pub/sub | Python |

---

## Blockchain

**`block_chain/`**

Connects to the Bitcoin P2P network and retrieves a specific block using the Bitcoin protocol over TCP.

- Implements Bitcoin wire protocol (version handshake, block fetch)
- Decodes and displays block headers, transactions, and merkle roots
- Uses compact size encoding for variable-length fields

**How to run:**
```bash
cd block_chain
python3 lab5.py
```

**Reference:** [Bitcoin Protocol](https://en.bitcoin.it/wiki/Protocol_documentation)

---

## Bully Algorithm

**`bully_algorithm/`**

Multi-threaded simulation of the Bully Algorithm for leader election in a distributed system.

- Creates 3 nodes that connect to a Group Coordinator Daemon (GCD)
- Nodes elect a leader via the Bully Algorithm
- Leader failure detection via async probing (0.5–3s intervals)
- Feigned failure simulation (leader drops requests temporarily)

**How to run:**
```bash
# Start GCD first (requires gcd2.py from course materials)
python3 gcd2.py <port>

# Start nodes
python3 bully_algo.py <host> <start_port>
```

**Reference:** [Bully Algorithm](https://en.wikipedia.org/wiki/Bully_algorithm)

---

## Chord DHT

**`chord_dht/`**

Distributed Hash Table implementation using the Chord protocol. Supports distributed key-value storage with efficient O(log n) lookup.

- Uses consistent hashing (SHA-1) to assign keys to nodes
- Maintains finger tables for O(log n) routing
- Nodes join/leave with proper successor/predecessor maintenance
- Tested with `Career_Stats_Passing.csv` data

**How to run:**
```bash
cd chord_dht

# Start chord nodes
python3 chord_node.py <port>

# Populate the DHT with data
python3 chord_populate.py <host> <port>

# Query the DHT
python3 chord_query.py <host> <port> <key>
```

**Reference:** [Chord Protocol (MIT)](https://pdos.csail.mit.edu/papers/chord:sigcomm01/chord_sigcomm.pdf)

---

## Real-Time Forex Arbitrage Detector

**`real_time_forex_arbitrage_detector/`**

Detects arbitrage opportunities in the foreign exchange market in real time using graph-based algorithms.

- Subscribes to live forex price feed via UDP
- Builds a currency exchange rate graph
- Detects negative cycles using Bellman-Ford (negative cycle = arbitrage opportunity)
- Prices expire after 1.5 seconds to reflect real-time validity

**How to run:**
```bash
cd real_time_forex_arbitrage_detector

# Start forex provider (requires course forex_provider.py)
python3 forex_provider.py

# Run arbitrage detector
python3 lab3.py <forex_provider_host> <forex_provider_port>
```

**Reference:** [Bellman-Ford Algorithm](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm) · [Forex Arbitrage](https://en.wikipedia.org/wiki/Triangular_arbitrage)

---

## Requirements

- Python 3.8+
- Standard library only (no external dependencies)

---

## Author

Benjamin Philipose · Seattle University CPSC 5520
