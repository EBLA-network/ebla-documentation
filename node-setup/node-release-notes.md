---
description: EBLA node release notes
---

# 🔢 Node Release Notes

## v2.1.0

_Storage performance & operator controls (DB Roadmap v01)_

**Docker image:** `docker pull ghcr.io/ebla-network/ebla-node:v2.1.0`

**Release Highlights:**

* **RocksDB 10** storage engine, with ZSTD + Snappy compression.
* **Hot/cold tiered storage** (opt-in, off by default) — recent data on fast disk, cold data archived to a cheaper tier.
* **Configurable memory** — bounded block cache and MemTable write-buffer budget for stable long-running validators.
* **New `--db-*` CLI flags + `EBLA_DB_*` env vars** — block-cache size, write-buffer size, max open files, tiering toggle, archive path, hot-size limit, cold-compression level (accept GB/MB/KB and GiB/MiB/KiB).

## v2.0.0

_First EBLA release — the full BlockDAG + PBFT node with EBLA's consensus, economics, and platform parameters._

**Docker image:** `docker pull ghcr.io/ebla-network/ebla-node:v2.0.0`

**Release Highlights:**

#### Consensus

* PBFT quorum threshold of **5/8** of the committee (15 of 24).
* Committee size 1,000; 20 block proposers; 50 DAG blocks per PBFT period.

#### Staking & economics (DPoS)

* **12,000,000,000 EBLA** hard supply cap.
* Initial staking yield **7%**, decaying epoch-by-epoch to a **1%** floor.
* Minimum validator commission **10%**, with a per-window rate-of-change cap.
* Minimum self-delegation **100 EBLA** to register; maximum validator stake **1,000,000 EBLA**.
* Inactivity slashing via voting-power decay, with automatic eviction below the **5,000 EBLA** eligibility threshold.

#### JSON-RPC & EVM

* **`ebla_*`** JSON-RPC namespace.
* Modernized EVM (latest opcodes, including `MCOPY`).

#### Storage & configuration

* Optional **LZ4** storage compression.
* Unified genesis configuration across Mainnet, Testnet, and Devnet.

#### Network identity

* Chain IDs — Mainnet **60186**, Testnet **60187**, Devnet **60188**; native currency **EBLA** (18 decimals).
