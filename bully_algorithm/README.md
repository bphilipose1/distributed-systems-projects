# Bully Algorithm — Distributed Leader Election

Multi-threaded Python implementation of the Bully Algorithm for leader election in a distributed system.

## What It Does

Simulates a distributed system with 3 nodes. Each node:
1. Connects to a Group Coordinator Daemon (GCD) and registers
2. Receives the list of other members
3. Participates in leader election via the Bully Algorithm
4. Monitors the leader via async probing; triggers re-election if leader fails

## The Bully Algorithm

When a node detects the leader has failed:
1. It sends `ELECTION` messages to all nodes with higher IDs
2. If no response → it declares itself leader, sends `COORDINATOR` to all
3. If a higher-ID node responds with `OK` → that node takes over the election
4. Highest available ID always wins (hence "bully")

**Reference:** [Bully Algorithm (Wikipedia)](https://en.wikipedia.org/wiki/Bully_algorithm)

## Extra Credit Features

- **Async Probing** — Follower nodes probe the leader every 0.5–3 seconds. If no response, triggers election
- **Feigned Failure** — Leader node periodically simulates failure by dropping requests for 0–4 seconds, then recovers

## How to Run

```bash
# 1. Start GCD (requires gcd2.py from course materials)
python3 gcd2.py <port>

# 2. Start nodes
python3 bully_algo.py <host> <start_port>
```

All nodes run on the same host for simplicity. Messages and election events are printed to console.

## Notes

- Nodes use TCP sockets for communication
- Probing uses separate async threads per follower node
- The main function initiates a killed-leader scenario automatically for testing
