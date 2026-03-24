# Real-Time Forex Arbitrage Detector

Detects arbitrage opportunities in the foreign exchange market in real time using the Bellman-Ford algorithm on a live price graph.

## What It Does

1. Subscribes to a UDP forex price feed
2. Maintains a graph of current exchange rates between currency pairs
3. Prices expire after **1.5 seconds** to ensure real-time accuracy
4. After each price update, runs Bellman-Ford to detect negative cycles
5. A **negative cycle = arbitrage opportunity** (you can trade around the cycle and profit)

## Why Bellman-Ford?

Standard shortest path algorithms (Dijkstra) don't handle negative edges. Bellman-Ford does, and a negative-weight cycle in a log-transformed currency graph directly corresponds to an arbitrage opportunity.

**The transformation:**  
Exchange rates are converted to `log(rate)` and negated. A profitable arbitrage cycle will have a negative total weight — detectable as a negative cycle.

**Reference:** [Bellman-Ford Algorithm](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm) · [Triangular Arbitrage](https://en.wikipedia.org/wiki/Triangular_arbitrage)

## Files

| File | Purpose |
|------|---------|
| `lab3.py` | Main arbitrage detector (UDP subscriber + Bellman-Ford) |
| `bellman_ford.py` | Graph implementation with negative cycle detection |
| `fxp_bytes.py` | Binary serialization utilities for forex messages |
| `fxp_bytes_subscriber.py` | UDP message deserialization + address serialization |
| `forex_provider.py` | Simulated forex price feed (course-provided) |
| `output.txt` | Sample output from a previous run |

## How to Run

```bash
# Start the forex provider
python3 forex_provider.py

# Run the arbitrage detector
python3 lab3.py <forex_provider_host> <forex_provider_port>
```

## How It Works

```
Forex Provider (UDP)
       ↓
  Price Update Received
       ↓
  Update exchange rate graph
  (expire stale prices > 1.5s old)
       ↓
  Run Bellman-Ford from each currency
       ↓
  Negative cycle found? → Print arbitrage path + profit
```

## Notes

- Uses UDP for low-latency price feed subscription
- Price validity window is 1.5 seconds (configurable via `VALIDITY_PERIOD`)
- Tolerates floating point noise in cycle detection via `tolerance` parameter
