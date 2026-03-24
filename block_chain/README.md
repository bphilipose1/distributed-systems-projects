# Blockchain — Bitcoin Protocol Implementation

Connects directly to the Bitcoin P2P network and retrieves a specific block using the Bitcoin wire protocol over TCP.

## What It Does

1. Opens a TCP connection to a Bitcoin node (`5.14.1.135:8333`)
2. Performs the version handshake (sends `version`, receives `verack`)
3. Requests and retrieves a target block based on student ID: `SU_ID % 10000`
4. Decodes block headers and transactions from raw bytes

## Key Concepts

- **Bitcoin Wire Protocol** — Binary message format with magic bytes, command, length, checksum, payload
- **Compact Size Encoding** — Variable-length integer encoding used throughout Bitcoin messages
- **Block Structure** — Header (version, prev block hash, merkle root, time, bits, nonce) + transactions
- **Proof of Work** — Block hash must be below a target threshold (the `bits` field)

## Message Format

```
[4 bytes magic] [12 bytes command] [4 bytes length] [4 bytes checksum] [payload]
```

Magic bytes: `f9beb4d9` (Bitcoin mainnet)

## How to Run

```bash
python3 lab5.py
```

Connects to a live Bitcoin node. Requires internet access.

## Reference

- [Bitcoin Protocol Documentation](https://en.bitcoin.it/wiki/Protocol_documentation)
- [Bitcoin Block Structure](https://en.bitcoin.it/wiki/Block)
