# Chord DHT — Distributed Hash Table

Implementation of the Chord protocol for distributed key-value storage with O(log n) lookup.

## What It Does

Builds a peer-to-peer distributed hash table where:
- Each node is assigned a position on a circular identifier ring using SHA-1 hashing
- Keys are mapped to nodes using consistent hashing
- Lookups route through finger tables in O(log n) hops

## The Chord Protocol

**Consistent Hashing:**  
Both nodes and keys are hashed to an m-bit ID space (ring). A key `k` is stored on the first node whose ID is ≥ `k` (its *successor*).

**Finger Tables:**  
Each node maintains a table of O(log n) entries pointing to nodes at exponentially increasing distances around the ring. This enables fast O(log n) routing rather than O(n) linear lookup.

**Reference:** [Chord: A Scalable Peer-to-peer Lookup Service (MIT)](https://pdos.csail.mit.edu/papers/chord:sigcomm01/chord_sigcomm.pdf)

## Files

| File | Purpose |
|------|---------|
| `chord_node.py` | Core node implementation (finger table, join, lookup) |
| `chord_populate.py` | Inserts key-value data into the DHT |
| `chord_query.py` | Queries a key from the DHT |
| `Career_Stats_Passing.csv` | Sample dataset used for testing |

## How to Run

```bash
# Start a chord node
python3 chord_node.py <port>

# Populate DHT with CSV data
python3 chord_populate.py <host> <port>

# Query a key
python3 chord_query.py <host> <port> <key>
```

## Configuration

```python
M = 7                  # Ring size: 2^M = 128 node slots
BUF_SZ = 4096          # Socket buffer size
TEST_PORTS = (34000, 34001, ...)  # Pre-verified non-conflicting ports at M=7
```

## Notes

- `TEST_PORTS` are manually verified to not cause hash ring conflicts at M=7
- Nodes use TCP + pickle for message serialization
- Allow ongoing protocols to finish before issuing new commands
